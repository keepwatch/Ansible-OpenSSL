# Ansible-OpenSSL

Automation tasks for building a certificate authority, creating and signing client/server certificates, and
transforming certificates into different formats.

## How to install

Scripts will be compiled and placed into the ../dist directory of the repo. The contents of this folder must be saved to your Ansible module library location (default /usr/lib/ansible) - see [here](http://docs.ansible.com/ansible/intro_configuration.html#library) for the config file entry.

**INSTALL**

`sh build.sh`

**What they hell are you building?**

In order to test the Python module outside of Ansible, you need to remove the module references.  Unfortunately, you can import anything that's not already available in the host machine's Python environment.  `source/build.py` simply replaces an import statement in the Ansible modules with the appropriate library (e.g. `from certificate import Certificate` gets replaced with the contents of `source/certificate.py`.

## How to use

First you need to setup a CA in the specific format these scripts are expecting:

```yaml
---

- name: Setup a CA
  ca: certdir="/etc/certs" subj="/DC=com/DC=example/CN=CA/"

```

Where, `certdir` is the root of the CA (you are responsible for making it) and `subj` is the valid OpenSSL subject of your CA.

To create a certificate, you simply need to reference the CA and provide the essential configuration:

```yaml
---

- name: Create a Server Cert
  certificate: cadir="/etc/certs" certname="server.example.com" subj="/DC=com/DC=example/CN=server/" p12password="{{some_env_var}}"

- name: Create a Client Cert
  certificate: cadir="/etc/certs" certname="client.example.com" subj="/DC=com/DC=example/CN=client/" p12password="{{some_env_var}}" certtype="client"

```

Additionally, the certificate module can use the parameter **days**, with an integer specifying the number of days for which the certificate will be valid. The default is ~10 years (3653 days).

Note: the default `certtype` is **server**, so you will need to specify **client** if you want a client certificate.  The primary difference between server and client certs is that the certificate is generated with the `-extensions server_ca_extensions` or `-extensions client_ca_extensions`, respectively.  The certificates will also be located in the appropriate directory (`server` or `client`) in the root of the CA directory.  The following files are created during this process:

1.  `{{certname}}.key.pem` - the private key.
2.  `{{certname}}.req.pem` - the signing request.
3.  `{{certname}}.cert.pem` - the certificate signed by the CA.
4.  `{{certname}}.keycert.pem` - a file containing both the private key and certificate (we found this necessary for some applications using SSL).
5.  `{{certname}}.keycert.p12` - the PKCS12 version of the key + cert pair.

It is your responsibility to move the certificates in and out of the CA directory.  This script assume you will perform actions to protect the CA.

To remove a certificate you need to specify the `state` of the cert:

```yaml
---

- name: Remove a Server Cert
  certificate: cadir="/etc/certs" certname="server.example.com" subj="doesn't matter" p12password="blah!" state="absent"

```

This will not only delete the certificate, the private key, certificate request, PKCS12 file, and key-cert, but it will also revoke the certificate from the authority.

Deleting the CA functions similarly:

```yaml
---

- name: Remove a CA
  ca: certdir="/etc/certs" subj="/CN=whatever/" state="absent"

```

This will cause the CA directory to be deleted (the script doesn't care that it didn't create the directory, so suck it up).

Finally, it's worth mentioning that there is another tool to generate a Java Truststore from an OpenSSL PEM.

## License

Copyright (c) 2015 Richard Clayton

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.  IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.


