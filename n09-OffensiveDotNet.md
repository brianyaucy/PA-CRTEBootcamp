# Offensive .NET

- [Offensive .NET](#offensive-net)
  - [Offensive .NET Introduction](#offensive-net-introduction)
  - [Tradecraft](#tradecraft)

---

## Offensive .NET Introduction

Currently, `.NET` lacks some of the security features implemeneted in `System.Management.Automation.dll`. Because of this, many Red teams have included `.NET` in their tradecraft. 

There are many open source Offensive .NET tools and we will use the ones that fit out attack methodology.

<br/>

## Tradecraft

When using `.NET` (or any other complied language), there are some challenges:

1. Detection by countermeasures like AV, EDR, etc.
2. Delivery of the payload (recall PS sweet download-execute cradles!)
3. Detection by logging like **Process Creation**, **Command-line logging**, etc.

