.. index::
    single: Cache; Varnish

Come usare Varnish per accelerare il proprio sito
=================================================

Poiché la cache di Symfony2 usa gli header standard della cache HTTP,
:ref:`symfony-gateway-cache` può essere facilmente sostituito da qualsiasi altro reverse
proxy. Varnish è un acceleratore HTTP potente e open source, che è in grado di servire
contenuti in cache in modo veloce e che include il supporto per :ref:`Edge Side
Include<edge-side-includes>`.

.. index::
    single: Varnish; Configurazione

Configurazione
--------------

Come visto in precedenza, Symfony2 è abbastanza intelligente da capire se sta parlando
a un reverse proxy che capisca ESI o meno. Funziona immediatamente, se si usa il reverse
proxy di Symfony2, mentre occorre una configurazione speciale per poter funzionare
con Varnish. Fortunatamente, Symfony2 si appoggia a uno standard scritto
da Akamaï (`Architettura Edge`_), quindi i suggerimenti di configurazione in questo
capitolo possono essere utili anche non usando Symfony2.

.. note::

    Varnish supporta solo l'attributo ``src`` per i tag ESI (``onerror`` e
    ``alt`` vengono ignorati).

Prima di tutto, configurare Varnish in modo che pubblicizzi il suo supporto a ESI,
aggiungendo un header ``Surrogate-Capability`` alle richieste girate all'applicazione
sottostante:

.. code-block:: text

    sub vcl_recv {
        // Aggiunge un header Surrogate-Capability per dichiarare il supporto a ESI.
        set req.http.Surrogate-Capability = "abc=ESI/1.0";
    }

Quindi, ottimizzare Varnish in modo che analizzi i contenuti della risposta solo quando
ci sia almeno un tag ESI, verificando l'header ``Surrogate-Control``, che
Symfony2 aggiunge automaticamente:

.. code-block:: text

    sub vcl_fetch {
        /* 
        Verifica il riconoscimento di ESI  
        e rimuove l'header Surrogate-Control
        */
        if (beresp.http.Surrogate-Control ~ "ESI/1.0") {
            unset beresp.http.Surrogate-Control;

            // per Varnish >= 3.0
            set beresp.do_esi = true;
            // per Varnish < 3.0
            // esi;
        }
    }

.. caution::

    La compressione con ESI non era supportata in Varnish fino alle versione 3.0
    (leggere `GZIP e Varnish`_). Se non si usa Varnish 3.0, inserire un server web
    davanti a Varnish per eseguire la compressione.

.. index::
    single: Varnish; Invalidare

Invalidare la cache
-------------------

Non si dovrebbe aver mai bisogno di invalidare dati in cache, perché l'invalidazione è
già gestita nativamente nei modelli di cache HTTP (vedere :ref:`http-cache-invalidation`).

Tuttavia, Varnish può essere configurato per accettare un metodo HTTP speciale ``PURGE``,
che invalida la cache per una data risorsa:

.. code-block:: text

    /* 
     Connessione al server di backend
     sulla porta 8080 della macchina locale
     */
    backend default {
        .host = "127.0.0.1";
        .port = "8080";
    }

    sub vcl_recv {
        /* 
        Il comportamento predefinito di Varnish non supporta PURGE.
        Individua le richieste PURGE e fa immediatamente una ricerca in cache, 
        altrimenti Varnish girerebbe direttamente la richiesta al backend
        e aggirerebbe la cache        
        */
        if (req.request == "PURGE") {
            return(lookup);
        }
    }

    sub vcl_hit {
        // Individua la richiesta PURGE
        if (req.request == "PURGE") {
            // Forza la scadenza dell'oggetto per Varnish < 3.0
            set obj.ttl = 0s;
            // Fa un vero purge per Varnish >= 3.0
            // purge;
            error 200 "Purged";
        }
    }

    sub vcl_miss {
        /*
        Individua la richiesta PURGE e
        indica che la richiesta non è stata messa in cache.
        */
        if (req.request == "PURGE") {
            error 404 "Not purged";
        }
    }

.. caution::

    Bisogna proteggere il metodo HTTP ``PURGE`` in qualche modo, per evitare che qualcuno
    pulisca i dati in cache in modo casuale.

    .. code-block:: text

        /* 
         Connessione al server di backend
         sulla porta 8080 della macchina locale
         */
        backend default {
            .host = "127.0.0.1";
            .port = "8080";
        }

        // Le acl possono contenere IP, sottoreti e nomi di host
        acl purge {
            "localhost";
            "192.168.55.0"/24;
        }

        sub vcl_recv {
            // Individua la richiesta PURGE per evitare l'aggiramento della cache
            if (req.request == "PURGE") {
                // Individua l'IP del client corrispondente all'acl
                if (!client.ip ~ purge) {
                    // Accesso negato
                    error 405 "Not allowed.";
                }
                // Esegue una ricerca in cache
                return(lookup);
            }
        }

        sub vcl_hit {
            // Individua la richiesta PURGE
            if (req.request == "PURGE") {
                // Forza la scadenza dell'oggetto per Varnish < 3.0
                set obj.ttl = 0s;
                // Fa un vero purge per Varnish >= 3.0
                // purge;
                error 200 "Purged";
            }
        }

        sub vcl_miss {
            // Individua la richiesta PURGE
            if (req.request == "PURGE") {
                // Indica che l'oggetto non è in cache
                error 404 "Not purged";
            }
        }

.. _`Architettura Edge`: http://www.w3.org/TR/edge-arch
.. _`GZIP e Varnish`:  https://www.varnish-cache.org/docs/3.0/phk/gzip.html