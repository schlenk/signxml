SignXML: XML Signature in Python
================================

*SignXML* is an implementation of the W3C `XML Signature <http://en.wikipedia.org/wiki/XML_Signature>`_ standard
in Python (both `"Second Edition" <http://www.w3.org/TR/xmldsig-core/>`_ and `Version 1.1
<http://www.w3.org/TR/xmldsig-core1/>`_). This standard (also known as XMLDSig and
`RFC 3275 <http://www.ietf.org/rfc/rfc3275.txt>`_) is used to provide payload security in
`SAML 2.0 <http://en.wikipedia.org/wiki/SAML_2.0>`_, among other uses. *SignXML* implements all of the required
components of the standard, and most recommended ones. Its features are:

* Use of `defusedxml.lxml <https://bitbucket.org/tiran/defusedxml>`_ to defend against common XML-based attacks when verifying signatures
* Extensions to allow signing with and verifying X.509 certificate chains, including hostname/CN validation
* Support for exclusive XML canonicalization with inclusive prefixes (`InclusiveNamespaces PrefixList <http://www.w3.org/TR/xml-exc-c14n/#def-InclusiveNamespaces-PrefixList>`_, required to verify signatures generated by some SAML implementations)
* Modern Python compatibility (2.7-3.4+ and PyPy)
* Well-supported, portable, reliable dependencies: `lxml <https://github.com/lxml/lxml>`_, `defusedxml <https://bitbucket.org/tiran/defusedxml>`_, `cryptography <https://github.com/pyca/cryptography>`_, `eight <https://github.com/kislyuk/eight>`_, `pyOpenSSL <https://github.com/pyca/pyopenssl>`_
* Comprehensive testing (including the XMLDSig interoperability suite) and `continuous integration <https://travis-ci.org/kislyuk/signxml>`_
* Simple interface with useful defaults
* Compactness, readability, and extensibility

Installation
------------
::

    pip install signxml

Note: SignXML depends on `lxml <https://github.com/lxml/lxml>`_ and `cryptography <https://github.com/pyca/cryptography>`_, which in turn depend on `OpenSSL <https://www.openssl.org/>`_, `LibXML <http://xmlsoft.org/>`_, and Python tools to interface with them. On Ubuntu, you can install those with::

    apt-get install python-dev libxml2-dev libxslt1-dev libssl-dev python-cffi

**Ubuntu 12.04:** ``python-cffi`` is not available on 12.04. Use ``apt-get install libffi-dev`` followed by ``pip install cffi``.

Synopsis
--------

SignXML uses the ElementTree API (also supported by lxml) to work with XML data.

.. code-block:: python

    from signxml import xmldsig

    cert = open("example.pem").read()
    key = open("example.key").read()
    root = ElementTree.fromstring(signature_data)
    xmldsig(root).sign(key=key, cert=cert)
    verified_data = xmldsig(root).verify()

Verifying SAML assertions
~~~~~~~~~~~~~~~~~~~~~~~~~

Assuming ``metadata.xml`` contains SAML metadata for the assertion source:

.. code-block:: python

    from lxml import etree
    from base64 import b64decode
    from signxml import xmldsig

    with open("metadata.xml", "rb") as fh:
        cert = etree.parse(fh).find("//ds:X509Certificate").text

    assertion_data = xmldsig(b64decode(assertion_body)).verify(x509_cert=cert)

.. admonition:: Signing SAML assertions

 The SAML assertion schema specifies a location for the enveloped XML signature (between ``<Issuer>`` and
 ``<Subject>``). To sign a SAML assertion in a schema-compliant way, insert a signature placeholder tag at that location
 before calling xmldsig: ``<ds:Signature Id="placeholder"></ds:Signature>``.

.. admonition:: See what is signed

 It is important to understand and follow the best practice rule of "See what is signed" when verifying XML
 signatures. The gist of this rule is: if your application neglects to verify that the information it trusts is
 what was actually signed, the attacker can supply a valid signature but point you to malicious data that wasn't signed
 by that signature.

 In SignXML, you can ensure that the information signed is what you expect to be signed by only trusting the
 data returned by the ``verify()`` method. The return value is the XML node or string that was signed.

 **Recommended reading:** http://www.w3.org/TR/xmldsig-bestpractices/#practices-applications

Detached signatures
~~~~~~~~~~~~~~~~~~~

The XML Signature specification requires support of detached signatures, where the signature document refers (in
``<Reference URI="...">``) to an external document. SignXML does not support generating detached signatures. To verify
a detached signature, pass a resolver callable to the ``xmldsig.verify()`` method.

See the `API documentation <https://signxml.readthedocs.org/en/latest/#id3>`_ for more.

Authors
-------
* Andrey Kislyuk

Links
-----
* `Project home page (GitHub) <https://github.com/kislyuk/signxml>`_
* `Documentation (Read the Docs) <https://signxml.readthedocs.org/en/latest/>`_
* `Package distribution <https://warehouse.python.org/project/signxml/>`_ `(PyPI) <https://pypi.python.org/pypi/signxml>`_
* `List of W3C XML Signature standards and drafts <http://www.w3.org/TR/#tr_XML_Signature>`_
* `W3C Recommendation: XML Signature Syntax and Processing (Second Edition) <http://www.w3.org/TR/xmldsig-core/>`_
* `W3C Recommendation: XML Signature Syntax and Processing Version 1.1 <http://www.w3.org/TR/xmldsig-core1>`_
* `W3C Working Group Note: XML Signature Syntax and Processing Version 2.0 <http://www.w3.org/TR/xmldsig-core2>`_
* `W3C Working Group Note: XML Signature Best Practices <http://www.w3.org/TR/xmldsig-bestpractices/>`_
* `XML-Signature Interoperability <http://www.w3.org/Signature/2001/04/05-xmldsig-interop.html>`_
* `W3C Working Group Note: Test Cases for C14N 1.1 and XMLDSig Interoperability <http://www.w3.org/TR/xmldsig2ed-tests/>`_
* `XMLSec: Related links <https://www.aleksey.com/xmlsec/related.html>`_

Bugs
~~~~
Please report bugs, issues, feature requests, etc. on `GitHub <https://github.com/kislyuk/signxml/issues>`_.

License
-------
Licensed under the terms of the `Apache License, Version 2.0 <http://www.apache.org/licenses/LICENSE-2.0>`_.

.. image:: https://travis-ci.org/kislyuk/signxml.svg
        :target: https://travis-ci.org/kislyuk/signxml
.. image:: https://coveralls.io/repos/kislyuk/signxml/badge.svg?branch=master
        :target: https://coveralls.io/r/kislyuk/signxml?branch=master
.. image:: https://pypip.in/version/signxml/badge.svg
        :target: https://pypi.python.org/pypi/signxml
.. image:: https://pypip.in/download/signxml/badge.svg
        :target: https://pypi.python.org/pypi/signxml
.. image:: https://pypip.in/py_versions/signxml/badge.svg
        :target: https://pypi.python.org/pypi/signxml
.. image:: https://readthedocs.org/projects/signxml/badge/?version=latest
        :target: https://signxml.readthedocs.org/
