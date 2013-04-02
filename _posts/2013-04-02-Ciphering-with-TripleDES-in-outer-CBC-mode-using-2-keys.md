---
layout: post
title: Ciphering with TripleDES in outer CBC mode using 2 keys
---

Recently, I was working on a project to implement the 3GPP TS 23.048 specification. It's a specification which deals with security
mechanisms for the (U)SIM application toolkit. Under a particular section (KIC and KID), I have to implement the Triple DES in outer CBC mode with 2 keys. Well, I initially implemented using python/Crypto library. Later I implemented it using Go.

Triple DES is a block cipher/encryption algorithm. In other words, applying DES algorithm 3 times on each block.
Generally we have to use 3 keys, but current sim cards supports only 2 keys (or maybe the sim card which I used was following 2 keys standard).
Enough with theory, lets get into action.

    package main

    import (
        "log"
        "fmt"
        "crypto/des"
        "crypto/cipher"
        "encoding/hex"
    )

    func main(){
        kic := "7962D9ECE03D1ACD4C76089DCE131543" //its a 16 bytes key (values in hex)
        initialVector := "0000000000000000" //8 bytes
        plainText := "112233445566778811223344556677881122334455667788" // length must be multiply of 8

        //convert the kic to 3 keys (k1, k2, k3 where k1 != k2 and k1 == k3)
        kic = fmt.Sprintf("%s%s", kic, kic[0:16])
    
        key, _ := hex.DecodeString(kic);
        iv, _ :=  hex.DecodeString(initialVector)
        src, _ := hex.DecodeString(plainText)

        dest := make([]byte, len(src))
        tdes, err := des.NewTripleDESCipher(key); if err != nil{
            log.Fatal(err)
        }
        encrypt := cipher.NewCBCEncrypter(tdes, iv)
        encrypt.CryptBlocks(dest, src)
    
        log.Println("Plain  value :", plainText)
        log.Println("Cipher value :", hex.EncodeToString(dest))
    }

Thats it, using standard Go library we can implement the entire 3GPP TS 23.048 specification.