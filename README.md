### CDP Local Repos + Ambari Headless Install

Bem-vindo à documentação do projeto para instalação automatizada de cluster via Ambari utilizando repositórios locais e VDF customizado (CDP-VDF.xml). Este guia descreve a arquitetura, o preparo dos repositórios HTTP internos, a adaptação mínima do VDF de ODP para CDP, e o fluxo 100% headless via API/Blueprint.

Esta página é a base do GitHub Pages do projeto. Seções que ainda não estiverem prontas trazem conteúdo placeholder.

#### Sumário
- Visão Geral
- Arquitetura e Premissas
- Repositórios Locais (HTTP)
- CDP-VDF.xml (a partir de ODP-VDF.xml)
- Ambari Headless: API, Blueprint, Host Mapping
- Execução End-to-End (Ansible)
- Troubleshooting
- Roadmap
- Referências

---

### Visão Geral

Objetivo:
- Publicar repositórios locais HTTP servidos por httpd na master do cluster.
- Reaproveitar o VDF existente (ODP-VDF.xml) realizando alterações mínimas para CDP (identidade e fontes).
- Provisionar e instalar um cluster via Ambari sem acesso à UI (somente API/Blueprint).

Pontos-chave:
- O Blueprint não referencia diretamente URLs de pacotes; isso vem do VDF e/ou overrides de repositório via API.
- A automação é feita com Ansible executando na master.
- Artefatos binários vêm via Camber (stash/CLI), baixados localmente, e publicados sob /var/www/html/repos.

---

### Arquitetura e Premissas

- Hostnames:
  - master.cdp (Ambari Server, httpd, Camber CLI, Ansible)
  - node1.cdp
  - node2.cdp
  - node3.cdp

- Sistema operacional: RedHat-like 9 (family="redhat9" no VDF).
- Portas:
  - 80 (HTTP repositórios)
  - 8080 (Ambari API)
  - 22 (SSH Ansible)
- Ferramentas:
  - httpd (Apache)
  - createrepo_c
  - Ambari Server/Agent
  - Ansible
  - Camber CLI

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Integer vel ipsum vel nibh placerat varius.

---

### Repositórios Locais (HTTP)

Estrutura de diretórios proposta:
- Release:
  - Local: `/var/www/html/repos/cdp-release/1.2.2.0-128/rpms`
  - URL: `http://master.cdp/repos/cdp-release/1.2.2.0-128/rpms`
- Utils:
  - Local: `/var/www/html/repos/cdp-utils/1.2.2.0/rpms`
  - URL: `http://master.cdp/repos/cdp-utils/1.2.2.0/rpms`

Passos resumidos:
1) Instalar httpd e createrepo_c:
   - `dnf install -y httpd createrepo_c && systemctl enable --now httpd`
2) Criar diretórios conforme acima.
3) Baixar artefatos com Camber para `/opt/staging` e extraí-los nos diretórios `rpms`.
4) Gerar metadados:
   - `createrepo_c --update /var/www/html/repos/cdp-release/1.2.2.0-128/rpms`
   - `createrepo_c --update /var/www/html/repos/cdp-utils/1.2.2.0/rpms`
5) Testar:
   - `curl -I http://master.cdp/repos/cdp-release/1.2.2.0-128/rpms/repodata/repomd.xml`
   - `curl -I http://master.cdp/repos/cdp-utils/1.2.2.0/rpms/repodata/repomd.xml`

Notas:
- Ajustar SELinux e firewall conforme necessário.
- Os nomes “cdp-release” e “cdp-utils” existem para clareza, mas podem manter “odp-*” se desejar zero impacto em ferramentas dependentes.

Lorem ipsum dolor sit amet, consectetur adipiscing elit.

---

### CDP-VDF.xml (a partir de ODP-VDF.xml)

Partimos do VDF fornecido (ODP-VDF.xml) e aplicamos mudanças mínimas:
- Identidade:
  - `<stack-id>`: de `ODP-1.2` para `CDP-1.2`
  - `<display>`: de `ODP-1.2.2.0` para `CDP-1.2.2.0`
  - Manter `<version>1.2.2.0</version>` e `<build>128</build>`
- Repositórios:
  - `<baseurl>`: apontar para `http://master.cdp/...`
  - Manter `package-version` e `family="redhat9"`
  - Recomendação: usar `repoid/reponame` com prefixo `CDP` para refletir a identidade; se houver risco de acoplamento, manter os nomes ODP.

Exemplo de trecho ajustado:
```xml
<release>
  <type>STANDARD</type>
  <stack-id>CDP-1.2</stack-id>
  <version>1.2.2.0</version>
  <build>128</build>
  <compatible-with>1\.\d+\.\d+\.\d+</compatible-with>
  <release-notes>https://example.com/docs/cdp-vdf</release-notes>
  <display>CDP-1.2.2.0</display>
</release>

<repository-info>
  <os family="redhat9">
    <package-version>1_2_2_0_128</package-version>
    <repo>
      <baseurl>http://master.cdp/repos/cdp-release/1.2.2.0-128/rpms</baseurl>
      <repoid>CDP-</repoid>
      <reponame>CDP</reponame>
      <unique>true</unique>
    </repo>
    <repo>
      <baseurl>http://master.cdp/repos/cdp-utils/1.2.2.0/rpms</baseurl>
      <repoid>CDP-UTILS-1.2.2.0</repoid>
      <reponame>CDP-UTILS</reponame>
      <unique>false</unique>
    </repo>
  </os>
</repository-info>
```

Nota de sanidade:
- O ODP-VDF.xml original contém uma linha potencialmente inconsistente:
  - `<service id="ZEPPELIN-0101" name="NIFI_REGISTRY" version="0.10.1"/>`
- Se você deseja mudanças mínimas, pode manter. Caso queira corrigir, ajuste para o serviço correto (ZEPPELIN 0.10.1) ou remova se não for necessário.

Lorem ipsum dolor sit amet.

---

### Ambari Headless: API, Blueprint, Host Mapping

Fluxo:
1) Instalar Ambari Server na master e Agents em todos os nós.
2) Registrar o VDF (CDP-VDF.xml) via API.
3) Registrar Blueprint via `POST /api/v1/blueprints/:name`.
4) Criar Cluster via `POST /api/v1/clusters/:clusterName` com host mapping.

Exemplo de chamadas:
```bash
# Registrar blueprint
curl -u admin:admin -H "X-Requested-By: ambari" \
  -H "Content-Type: application/json" \
  -X POST http://master.cdp:8080/api/v1/blueprints/cdp-small \
  -d @blueprint.json

# Criar cluster
curl -u admin:admin -H "X-Requested-By: ambari" \
  -H "Content-Type: application/json" \
  -X POST http://master.cdp:8080/api/v1/clusters/cdp-cluster \
  -d @host-mapping.json
```

Blueprint e Host Mapping:
- `stack_name`/`stack_version` alinhados com `<stack-id>CDP-1.2`.
- Evite `repository_settings` se o VDF já cobre os repos.

Lorem ipsum dolor sit amet, consectetur adipiscing elit.

---

### Execução End-to-End (Ansible)

Ordem sugerida de playbooks:
1) base-tools.yml — instala httpd, createrepo_c, utilitários.
2) camber-sync.yml — baixa artefatos e atualiza `/var/www/html/repos/.../rpms`.
3) ambari-install.yml — instala e inicia Ambari Server/Agent.
4) vdf-register.yml — importa o CDP-VDF.xml via API.
5) blueprint-register.yml — registra o blueprint.
6) create-cluster.yml — cria o cluster e acompanha requests.

Variáveis:
- `repo_base_host: master.cdp`
- `release_path: /var/www/html/repos/cdp-release/1.2.2.0-128/rpms`
- `utils_path: /var/www/html/repos/cdp-utils/1.2.2.0/rpms`
- Segredos do Camber por Ansible Vault.

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Mauris at justo id dui.

---

### Troubleshooting

- HTTP 404 no repodata:
  - Conferir caminhos e permissões, rodar `createrepo_c`, SELinux labels.
- verify_base_url falha no Ambari:
  - Testar acesso dos nodes ao `baseurl`, DNS e firewall.
- Agent não registra:
  - `ambari-agent.ini` apontando para `master.cdp`, checar clock/NTP.
- Erros de instalação de pacotes:
  - Confirmar que todos os pacotes estão presentes nos repos release/utils.

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Vivamus aliquet.

---

### Roadmap

- Assinatura GPG dos repositórios e `gpgcheck=1`.
- Health-checks e observabilidade.
- HA para HDFS/YARN/ZK.
- Rotina de re-sync do Camber com checagem de integridade.

Lorem ipsum dolor sit amet, consectetur adipiscing elit.

---

### Referências

- Ambari Blueprints: [Apache Ambari Blueprints](https://ambari.apache.org/docs/3.0.0/blueprints/)
- Repositórios locais: [Local Repos (HDP/Ambari)](https://docs.hortonworks.com/HDPDocuments/Ambari-2.7.3.0/bk_ambari-installation/content/setting_up_a_local_repository_with_temporary_internet_access.html)
- Seleção de versão e VDF: [Select Version (Ambari)](https://docs.cloudera.com/HDPDocuments/Ambari-2.6.2.2/bk_ambari-installation/content/select_version.html)
- Clemlab docs: Lorem ipsum link

---

Se quiser, posso:
- Entregar o arquivo CDP-VDF.xml completo com as alterações mínimas.
- Fornecer blueprint.json e host-mapping.json de exemplo.
- Criar um índice lateral (TOC) para o GitHub Pages com links para subpáginas (repos, vdf, blueprint, automação).