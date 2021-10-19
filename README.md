# Typescript template for OpenFaaS

This template is based on the official `node12` template from OpenFaaS. It allows you to write your function in Typescript, that will be compiled to javascript during the build process. This template uses openfaas/classic-watchdog

## Usage

```shell
faas-cli template pull https://github.com/vijetanagar/openfaas-template-classic-typescript
faas-cli new <my-typescript-function> --lang ts-classic-node
```
