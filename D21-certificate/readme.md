Steps:
    step 1: The new user has to generate the private key using CSR (Certificate Signing Request)
    
        openssl genrsa -out ramesh.key 2048

        Breakdown of the components:

        openssl: The command-line tool used for various Cryptography and SSL/TLS operations.
        
        genrsa: The specific sub-command that tells OpenSSL to generate an RSA private key.

        -out ramesh.key: This flag specifies the output filename. In this case, the generated private key will be saved to a file named ramesh.key.

        2048: This specifies the size of the key in bits. 2048 bits is currently the industry standard for security and performance; larger keys (like 4096) are more secure but computationally slower. 

    step 2: The new user has to create a Certificate Signing Request (CSR) using an existing private key.

        openssl req -new -key ramesh.key -out ramesh.csr -subj "/CN=ramesh"

        Here is a breakdown of each part:
        
        openssl req: Invokes the OpenSSL utility for managing PKCS#10 certificate requests.
        
        -new: Indicates that a new certificate request is being generated.
        
        -key ramesh.key: Specifies the input file containing the private key (created in the previous step) that will be used to sign the request and prove ownership.

        -out ramesh.csr: Specifies the filename for the resulting CSR. This file is what you would typically send to a Certificate Authority (CA) to get a signed SSL/TLS certificate.

        -subj "/CN=ramesh": Short for "subject." This flag provides the identification information for the certificate directly in the command line to avoid interactive prompts.

        /CN=ramesh: Sets the Common Name (CN) attribute to "ramesh." In a real-world scenario, this is usually the Fully Qualified Domain Name (FQDN), such as www.example.com, which the certificate will protect. 

        Once these 2 steps created, the new user gives the "ramesh.csr: file, we need to use this file in further steps
    
    step 3: we (existing k8s admin) needs to create a .yaml file (ex: ramesh-csr.yaml)
        a. The content of the file (ramesh.csr) in plain text.
        b. copy the value outeside and encode the value of file to base64 and remove all new lines to make it a single line value.

       cat ramesh.csr | base64 | tr -d "\n"

    step 4: schedule/apply this yaml

        kubuectl apply -f ramesh-csr.yaml

    step 5: Approve or deny the request

        kubectl certificate approve ramesh-sign-request 

    step 6: One the certificate is approved , share the certificte with user 

        kubectl get csr ramesh-sign-request -o yaml > ramesh-csr-approved.yaml
