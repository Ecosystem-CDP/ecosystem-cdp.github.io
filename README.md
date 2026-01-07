# Automação de Data Lake CDP na Oracle Cloud

Bem-vindo à documentação oficial do projeto de automação **CDP (Cloudera/Open Data Platform)** na **OCI**. Este projeto oferece uma solução "Infrastructure as Code" completa para implantar um cluster Hadoop seguro, escalável e pronto para produção em minutos.

![banner](https://img.shields.io/badge/Status-Stable-success) ![terraform](https://img.shields.io/badge/IaC-Terraform-purple) ![ansible](https://img.shields.io/badge/Config-Ansible-red) ![oci](https://img.shields.io/badge/Cloud-Oracle-orange)

---

##  Visão Geral

O objetivo deste projeto é eliminar a complexidade da instalação manual de clusters Big Data. Utilizando **Terraform** para infraestrutura e **Ansible** para configuração, entregamos um ambiente **Apache Ambari** totalmente funcional com serviços como HDFS, YARN, Hive, Spark e Ranger pré-configurados.

### Principais Diferenciais
*   **Zero-Touch Deployment**: De `terraform apply` até o cluster rodando sem intervenção humana.
*   **OCI Stacks**: Integração nativa com o Oracle Resource Manager para deploy via interface web.
*   **Segurança First**: Security Lists configuradas, chaves SSH gerenciadas e serviços internos isolados.

---

##  Arquitetura

O deploy provisiona uma VCN com subnets otimizadas e instâncias Compute (Ampere A1) interconectadas.

```mermaid
graph LR
    User((Usuário)) -->|OCI Stack| RM[Resource Manager]
    RM -->|Terraform| VCN[OCI VCN]
    
    subgraph Cluster Hadoop
        Master[Master Node<br>Ambari Server]
        W1[Worker 1]
        W2[Worker 2]
        W3[Worker 3]
    end
    
    VCN --> Master
    Master --> W1 & W2 & W3
```

---

## Documentação Completa

Aprofunde-se nos detalhes técnicos e operacionais através dos nossos guias:

###  Começando
*   **[Resumo Executivo](https://github.com/Ecosystem-CDP/docs/blob/main/docs/03%20-%20CDP/01-resumo-executivo.md)**: Entenda o escopo e os componentes da solução.
*   **[Guia de Deploy via OCI Stacks](https://github.com/Ecosystem-CDP/docs/blob/main/docs/03%20-%20CDP/07-guia-deploy-oci-stacks.md)**: **(Recomendado)** O passo a passo mais simples para subir seu cluster.

###  Técnico & Operação
*   **[Guia Técnico de Automação](https://github.com/Ecosystem-CDP/docs/blob/main/docs/03%20-%20CDP/02-guia-tecnico-automacao.md)**: Como o Terraform e Ansible trabalham juntos.
*   **[Arquitetura e Fluxo](https://github.com/Ecosystem-CDP/docs/blob/main/docs/03%20-%20CDP/04-arquitetura-fluxo.md)**: Diagramas detalhados do provisionamento.
*   **[Manual de Debug](https://github.com/Ecosystem-CDP/docs/blob/main/docs/03%20-%20CDP/03-manual-debug.md)**: O que fazer se algo der errado.

###  Referência
*   **[Credenciais e Senhas](https://github.com/Ecosystem-CDP/docs/blob/main/docs/03%20-%20CDP/05-senhas-credenciais.md)**: Acessos padrão do ambiente.
*   **[Scripts de Referência](https://github.com/Ecosystem-CDP/docs/blob/main/docs/03%20-%20CDP/06-scripts-referencia.md)**: Utilitários para o dia a dia.

---

## Contribuição

Este é um projeto Open Source. Sinta-se à vontade para abrir Issues ou Pull Requests no nosso repositório para melhorias na automação ou documentação.

---
*Gerado automaticamente para apresentação via GitHub Pages.*
