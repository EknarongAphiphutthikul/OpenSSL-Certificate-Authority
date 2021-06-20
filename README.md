# OpenSSL-Certificate-Authority
- Generate Root CA
  ```sh
  mkdir root-ca

  cd root-ca

  openssl genrsa -aes256 -passout pass:changeit -out ca.key.pem 4096

  chmod 400 ca.key.pem

  openssl req -passin pass:changeit -key ca.key.pem -new -x509 -days 7300 -sha256 -out ca.cert.pem -subj "/C=TH/ST=Bangkok/L=Phayathai/O=ake-demo/CN=AKE Root CA/"

  openssl x509 -noout -text -in ca.cert.pem
  ```

- Generate Intermediate CA
  - Generate
    ```sh 
    mkdir intermediate-ca

    cd intermediate-ca

    openssl genrsa -aes256 -passout pass:changeit -out intermediate.key.pem 4096

    chmod 400 intermediate.key.pem

    openssl req -passin pass:changeit -key intermediate.key.pem -new -sha256 -out intermediate.csr.pem -subj "/C=TH/ST=Bangkok/L=Phayathai/O=ake-demo/CN=AKE intermediate CA/"
    ```
  - Signed Root Certificate
    ```sh
    openssl x509 -req -in intermediate.csr.pem -CA ../root-ca/ca.cert.pem -CAkey ../root-ca/ca.key.pem -passin pass:changeit -CAcreateserial -days 3650 -sha256 -out intermediate.cert.pem

    OR

    openssl x509 -req -in intermediate.csr.pem -CA ../root-ca/ca.cert.pem -CAkey ../root-ca/ca.key.pem -passin pass:changeit -CAserial ../root-ca/ca.cert.srl -days 3650 -sha256 -out intermediate.cert.pem
    ```
  - verify
    ```sh
    openssl x509 -noout -text -in intermediate.cert.pem

    openssl verify -CAfile ../root-ca/ca.cert.pem intermediate.cert.pem
    ```
  - create certificate chain
    ```sh
    cat intermediate.cert.pem ../root-ca/ca.cert.pem > ca-chain.cert.pem
    ```