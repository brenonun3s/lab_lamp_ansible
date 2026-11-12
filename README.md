# Laboratório Ansible — Stack LAMP com WordPress

Laboratório de estudos em que um **Control Node** com Ansible configura, por SSH, dois servidores de distribuições diferentes (**Ubuntu Server** e **Rocky Linux**) com Apache, PHP, MariaDB e WordPress, usando as mesmas roles.

<p align="center">
  <img src="docs/images/arquitetura-lab-ansible.png" alt="Arquitetura do laboratório: Control Node com Ansible conectado por SSH a dois Managed Nodes, Ubuntu e Rocky Linux, com PHP, WordPress e MariaDB" width="800">
</p>

| Função | Sistema operacional | Endereço IP |
|---|---|---|
| Control Node | Ubuntu Server | `192.168.81.135` |
| Managed Node 01 | Ubuntu Server | `192.168.81.137` |
| Managed Node 02 | Rocky Linux | `192.168.81.144` |

> Ambiente em máquinas virtuais e rede local. Substitua os IPs ao reproduzir.

**Stack:** Ansible · Ubuntu Server · Rocky Linux · Apache · PHP-FPM · MariaDB · WordPress

## Destaques

- **Duas distribuições, uma automação:** tarefas separadas em `debian.yml` e `redhat.yml`, selecionadas por `ansible_facts['os_family']` (APT × DNF, `apache2` × `httpd`).
- **Roles, variáveis, templates e handlers:** o VirtualHost do Apache é gerado por template Jinja2, e o Apache só é recarregado quando a configuração muda.
- **Idempotência:** tarefas com `creates`/`removes` podem ser executadas várias vezes sem efeitos colaterais.
- **Troubleshooting na automação:** o Ubuntu exibia a página padrão do Apache por causa do `000-default.conf`. A causa foi corrigida no playbook, sem ajuste manual no servidor. [Ver o caso completo](docs/troubleshooting.md).

## Como executar

A partir do Control Node:

```bash
# Testa a comunicação com os hosts
ansible all -i inventory.ini -m ping

# Verifica a sintaxe
ansible-playbook -i inventory.ini site.yml --syntax-check

# Executa a automação
ansible-playbook -i inventory.ini site.yml
```

Validação em cada servidor:

```bash
curl -s http://localhost | grep -i "Lab Ansible"
```

## Estrutura

```text
.
├── inventory.ini
├── site.yml
├── docs/
│   ├── images/
│   └── troubleshooting.md
└── roles/
    ├── apache/        # Apache, VirtualHost e página HTML
    │   ├── files/
    │   ├── handlers/
    │   ├── tasks/     # main.yml, debian.yml, redhat.yml
    │   ├── templates/
    │   └── vars/
    ├── php/           # PHP-FPM
    ├── mysql/         # MariaDB
    └── serverTools/   # Pacotes e utilitários base
```

## Próximos passos

- Automatizar a instalação do WordPress em uma role dedicada.
- Separar variáveis por distribuição (ex.: `php_version`).
- Configurar HTTPS no Apache.
- Avançar em Terraform e Infrastructure as Code.

---

Projeto de estudos. Em ambientes reais, use autenticação SSH por chave e Ansible Vault para segredos.