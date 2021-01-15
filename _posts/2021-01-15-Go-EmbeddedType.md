---
title: "Go Embedded type"
date: 2021-01-15
---

# Go Embedded type
- [Struct types](https://golang.org/ref/spec#Struct_types)  
> Given a struct type S and a defined type T, promoted methods are included in the method set of the struct as follows:
> If S contains an embedded field T, the method sets of S and *S both include promoted methods with receiver T. The method set of *S also includes promoted methods with receiver *T.
> If S contains an embedded field *T, the method sets of S and *S both include promoted methods with receiver T or *T.
