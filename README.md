# OpenSSL-Certificate-Authority
- Generate Root CA
  - Generate
    ```sh
    cd root-ca
    mkdir certs crl newcerts private
    touch index.txt
    echo 1000 > serial

    openssl genrsa -aes256 -passout pass:changeit -out private/ca.key.pem 4096

    openssl req -config openssl.cnf -passin pass:changeit -key private/ca.key.pem -new -x509 -days 7300 -sha256 -extensions v3_ca -out certs/ca.cert.pem -subj "/C=TH/ST=Bangkok/L=Phayathai/O=ake-demo/CN=AKE Root CA/"

    openssl x509 -noout -text -in certs/ca.cert.pem
    ```

- Generate Intermediate CA
  - Generate
    ```sh 
    cd intermediate-ca
    mkdir certs crl csr newcerts private
    touch index.txt
    echo 1000 > serial
    echo 1000 > ../root-ca/crlnumber

    openssl genrsa -aes256 -passout pass:changeit -out private/intermediate.key.pem 4096

    openssl req -config openssl.cnf -passin pass:changeit -key private/intermediate.key.pem -new -sha256 -out csr/intermediate.csr.pem -subj "/C=TH/ST=Bangkok/L=Phayathai/O=ake-demo/CN=AKE intermediate CA/"
    ```
  - Signed Root Certificate
    ```sh
    cd ../root-ca/
    openssl ca -batch -config openssl.cnf -extensions v3_intermediate_ca -passin pass:changeit -days 3650 -notext -md sha256 -in ../intermediate-ca/csr/intermediate.csr.pem -out ../intermediate-ca/certs/intermediate.cert.pem
    ```
  - verify
    ```sh
    openssl x509 -noout -text -in ../intermediate-ca/certs/intermediate.cert.pem

    openssl verify -CAfile certs/ca.cert.pem ../intermediate-ca/certs/intermediate.cert.pem
    ```
  - create certificate chain
    ```sh
    cat ../intermediate-ca/certs/intermediate.cert.pem certs/ca.cert.pem > ../intermediate-ca/certs/ca-chain.cert.pem
    ```

- Generate Machine Cetificate
  - create folder
    ```sh
    mkdir machine
    mkdir machine/keystore
    ```
  - create truststore
    ```sh
    cd intermediate-ca

    keytool -keystore ../machine/keystore/truststore.jks -storetype PKCS12 -alias rootca -import -file ../root-ca/certs/ca.cert.pem -storepass changeit

    keytool -importcert -alias intermediateca -keystore ../machine/keystore/truststore.jks -file certs/intermediate.cert.pem -storepass changeit
    ```
  - create Git Server Cetificate
    ```sh 
    cd intermediate-ca
    openssl genrsa -aes256 -passout pass:changeit -out ../machine/gitserver.key.pem 2048

    openssl req -config openssl.cnf -passin pass:changeit -passout pass:changeit -key ../machine/gitserver.key.pem -new -sha256 -out ../machine/gitserver.csr.pem -subj "/C=TH/ST=Bangkok/L=Phayathai/O=ake-demo/CN=gitserver.ake.com/"

    openssl ca -batch -config openssl.cnf -extensions server_cert -passin pass:changeit -days 1825 -notext -md sha256 -in ../machine/gitserver.csr.pem -out ../machine/gitserver.cert.pem

    openssl x509 -noout -text -in ../machine/gitserver.cert.pem

    openssl verify -CAfile certs/ca-chain.cert.pem ../machine/gitserver.cert.pem

    openssl pkcs12 -export -passin pass:changeit -passout pass:changeit -out ../machine/keystore/gitserver.p12 -inkey ../machine/gitserver.key.pem -in ../machine/gitserver.cert.pem -certfile certs/ca-chain.cert.pem -name gitserver.ake.com
    ```
  - create Docker Registry Cetificate
    ```sh 
    cd intermediate-ca
    openssl genrsa -aes256 -passout pass:changeit -out ../machine/registry.key.pem 2048

    openssl req -config openssl.cnf -passin pass:changeit -passout pass:changeit -key ../machine/registry.key.pem -new -sha256 -out ../machine/registry.csr.pem -subj "/C=TH/ST=Bangkok/L=Phayathai/O=ake-demo/CN=registry.ake.com/"

    openssl ca -batch -config openssl.cnf -extensions server_cert -passin pass:changeit -days 1825 -notext -md sha256 -in ../machine/registry.csr.pem -out ../machine/registry.cert.pem

    openssl x509 -noout -text -in ../machine/registry.cert.pem

    openssl verify -CAfile certs/ca-chain.cert.pem ../machine/registry.cert.pem

    openssl pkcs12 -export -passin pass:changeit -passout pass:changeit -out ../machine/keystore/registry.p12 -inkey ../machine/registry.key.pem -in ../machine/registry.cert.pem -certfile certs/ca-chain.cert.pem -name registry.ake.com
    ```
  - create Jenkins Cetificate
    ```sh 
    cd intermediate-ca
    openssl genrsa -aes256 -passout pass:changeit -out ../machine/jenkins.key.pem 2048

    openssl req -config openssl.cnf -passin pass:changeit -passout pass:changeit -key ../machine/jenkins.key.pem -new -sha256 -out ../machine/jenkins.csr.pem -subj "/C=TH/ST=Bangkok/L=Phayathai/O=ake-demo/CN=jenkins.ake.com/"

    openssl ca -batch -config openssl.cnf -extensions server_cert -passin pass:changeit -days 1825 -notext -md sha256 -in ../machine/jenkins.csr.pem -out ../machine/jenkins.cert.pem

    openssl x509 -noout -text -in ../machine/jenkins.cert.pem

    openssl verify -CAfile certs/ca-chain.cert.pem ../machine/jenkins.cert.pem

    openssl pkcs12 -export -passin pass:changeit -passout pass:changeit -out ../machine/keystore/jenkins.p12 -inkey ../machine/jenkins.key.pem -in ../machine/jenkins.cert.pem -certfile certs/ca-chain.cert.pem -name jenkins.ake.com
    ```