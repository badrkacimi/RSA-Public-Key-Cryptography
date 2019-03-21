# RSA-Public-Key-Cryptography
RSA Public Key Cryptography in Java


RSA Public Key Cryptography in Java
Public key cryptography is a well-known concept, but for some reason the JCE (Java Cryptography Extensions) documentation doesn't at all make it clear how to interoperate with common public key formats such as those produced by openssl. If you try to do a search on the web for how to make RSA public key cryptography work in Java, you quickly find a lot of people asking questions and not a lot of people answering them. In this post, I'm going to try to lay out very clearly how I got this working.

Just to set expectations, this is not a tutorial about how to use the cryptography APIs themselves in javax.crypto (look at the JCE tutorials from Sun for this); nor is this a primer about how public key cryptography works. This article is really about how to manage the keys with off-the-shelf utilities available to your friendly, neighborhood sysadmin and still make use of them from Java programs. Really, this boils down to "how do I get these darn keys loaded into a Java program where they can be used?" This is the article I wish I had when I started trying to muck around with this stuff....

Managing the keys
Openssl. This is the de-facto tool sysadmins use for managing public/private keys, X.509 certificates, etc. This is what we want to create/manage our keys with, so that they can be stored in formats that are common across most Un*x systems and utilities (like, say, C programs using the openssl library...). Java has this notion of its own keystore, and Sun will give you the keytool command with Java, but that doesn't do you much good outside of Java world.

Creating the keypair. We are going to create a keypair, saving it in openssl's preferred PEM format. PEM formats are ASCII and hence easy to email around as needed. However, we will need to save the keys in the binary DER format so Java can read them. Without further ado, here is the magical incantation for creating the keys we'll use:

# generate a 2048-bit RSA private key
$ openssl genrsa -out private_key.pem 2048

# convert private Key to PKCS#8 format (so Java can read it)
$ openssl pkcs8 -topk8 -inform PEM -outform DER -in private_key.pem \
    -out private_key.der -nocrypt

# output public key portion in DER format (so Java can read it)
$ openssl rsa -in private_key.pem -pubout -outform DER -out public_key.der
You keep private_key.pem around for reference, but you hand the DER versions to your Java programs.

Loading the keys into Java
Really, this boils down to knowing what type of KeySpec to use when reading in the keys. To read in the private key:

import java.io.*;
import java.security.*;
import java.security.spec.*;

public class PrivateKeyReader {

  public static PrivateKey get(String filename)
    throws Exception {
    
    File f = new File(filename);
    FileInputStream fis = new FileInputStream(f);
    DataInputStream dis = new DataInputStream(fis);
    byte[] keyBytes = new byte[(int)f.length()];
    dis.readFully(keyBytes);
    dis.close();

    PKCS8EncodedKeySpec spec =
      new PKCS8EncodedKeySpec(keyBytes);
    KeyFactory kf = KeyFactory.getInstance("RSA");
    return kf.generatePrivate(spec);
  }
}
And now, to read in the public key:

import java.io.*;
import java.security.*;
import java.security.spec.*;

public class PublicKeyReader {

  public static PublicKey get(String filename)
    throws Exception {
    
    File f = new File(filename);
    FileInputStream fis = new FileInputStream(f);
    DataInputStream dis = new DataInputStream(fis);
    byte[] keyBytes = new byte[(int)f.length()];
    dis.readFully(keyBytes);
    dis.close();

    X509EncodedKeySpec spec =
      new X509EncodedKeySpec(keyBytes);
    KeyFactory kf = KeyFactory.getInstance("RSA");
    return kf.generatePublic(spec);
  }
}
That's about it. The hard part was figuring out a compatible set of:

openssl DER output options (particularly the PKCS#8 encoding)
which type of KeySpec Java needed to use (strangely enough, the public key needs the "X509" keyspec, even though you would normally handle X.509 certificates with the openssl x509 command, not the openssl rsa command. Real intuitive.)
From here, signing and verifying work as described in the JCE documentation; the only other thing you need to know is that you can use the "SHA1withRSA" algorithm when you get your java.security.Signature instance for signing/verifying, and that you want the "RSA" algorithm when you get your javax.crypto.Cipher instance for encrypting/decrypting.


For more visit :
http://codeartisan.blogspot.com/2009/05/public-key-cryptography-in-java.html


Many thanks to Jon Moore 
