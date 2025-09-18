-----------------------------------------------------
problem with self signed certificate - SSL inspection
-----------------------------------------------------

    echo | openssl s_client -connect apt.postgresql.org:443 -servername apt.postgresql.org -showcerts

* copy the certificate block (from -----BEGIN CERTIFICATE----- to -----END CERTIFICATE-----) for the proxy’s CA (often the second or third in the chain). Save it to a file (e.g., proxy.crt).

* Trust the Proxy’s CA Certificate
* Add the proxy’s CA certificate to your system’s trusted store
* Copy the certificate:

    sudo cp proxy1.crt /usr/local/share/ca-certificates/

* Update the CA store:

    sudo udate-ca-certificates

* update files on server

    sudo apt-get update