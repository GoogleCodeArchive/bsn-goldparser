# Tools and libraries #

  * VS 2010 (or compatible MSBUILD config for 2.0 target):
    * .NET 3.5
    * C# 3 compiler
  * [nUnit 2.5.3](http://www.nunit.org/?p=download) to compile and run the unit tests

# Strong name key #

The library and the unit tests are signed with a strong name key. For security reasons, I keep my strong name keys in the key container instead of files. Therefore, the key used by me is not included in this distribution.

In order to get yourself a key to enable the compilation, load a strong name key into the `bsn` key container using the `sn` utility:
```
sn -i <infile> bsn
```

If you need to create a strong name key file, also do this with this utility:
```
sn -k <outfile>
```

Be aware that your key is going to yield a different public key token on the assembly, so that you cannot build a version with an assembly name identical to the original binaries.

To make the `InternalsVisibleTo` work for the unit tests, you need the full public key of the assembly. You can get this with `sn` as well from the compiled library:
```
sn -Tp bsn.GoldParser.dll

Microsoft (R) .NET Framework Strong Name Utility  Version 3.5.30729.1
Copyright (c) Microsoft Corporation.  All rights reserved.

Public key is
0024000004800000140100000602000000240000525341310008000001000100f9f8bece3c34a4
e8cbe7d3c25e3fb8fa3cd50a0f0a78eba8cdc00d55479d75da366d5b2d57f15d2004e02477c140
e36c3fd897457c1fb3522cfcb9673a0a97beb154c95d3db73221acbaae480296792b7b435da63c
e18cb8cd5cc8fd3c827735e7627915d781a4e99ddb5815125c7fa2afd71938a8c07b89eb50c9b7
a6e1e716bc704bb6eb0dc3a68f778f552833cc405b042e3e2b56ae8bee322e3a450136ebc15c38
a636497f3b169c78d216f5d89bfcb2ac203fb60d2d4ca2685193b8a41a1a8b9f9871d782d7a7d3
d98600cf318ad269843cc11182558de704714abaa740c2bca3729ea43da545422302d8ddd27a4b
614902956da6d36dd195e719bc0ad6

Public key token is b7a8e330c2d95480
```
You need to put the full public key into that attribute, and the tests will compile and work as well.