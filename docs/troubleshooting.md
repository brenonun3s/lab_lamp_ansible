# Troubleshooting: página padrão do Apache no Ubuntu

[← Voltar ao README](../README.md)

## Sintoma

Depois de executar o playbook, o Rocky Linux exibia a página personalizada **Lab Ansible**, mas o Ubuntu mostrava a página padrão do Apache (`Apache2 Ubuntu Default Page`).

## Diagnóstico

**1. O que o Ubuntu estava servindo**

```bash
head -30 /var/www/html/index.html          # página padrão do Apache
grep -n "Lab Ansible" /var/www/html/index.html   # nenhum resultado
```

**2. Para onde a automação copiava o HTML**

A tarefa de cópia usa `dest: "{{ site_folder }}"`, definido como `/var/www/breno_local`, e não `/var/www/html`. O arquivo estava lá:

```bash
ls -lah /var/www/breno_local/
```

**3. Por que o Apache não entregava esse arquivo**

O VirtualHost do projeto responde por `ServerName breno.local`. Ao acessar pelo IP, nenhum `ServerName` corresponde, e o Apache entrega o primeiro VirtualHost carregado. No Ubuntu, esse é o `000-default.conf`, habilitado por padrão e apontando para `/var/www/html`. O Rocky não tem um VirtualHost padrão concorrente, por isso funcionava.

**4. Confirmação**

Simulando o acesso pelo nome do site:

```bash
curl -s -H "Host: breno.local" http://localhost | grep -i "Lab Ansible"
```

A página personalizada apareceu: a automação tinha funcionado, e o problema era apenas qual VirtualHost respondia ao IP.

## Correções na automação

**Desabilitar o site padrão no Ubuntu** (`roles/apache/tasks/debian.yml`):

```yaml
- name: "Disable Apache default site - Debian"
  ansible.builtin.command: a2dissite 000-default
  args:
    removes: /etc/apache2/sites-enabled/000-default.conf
  become: true
  when: (ansible_facts['os_family'] == "Debian")
  notify:
    - reloadApache2
```

**Criar o handler `enableSite`**, que era notificado pelo template do VirtualHost mas não existia (`roles/apache/handlers/main.yml`):

```yaml
- name: "enableSite"
  ansible.builtin.command: a2ensite {{ log_prefix }}
  args:
    creates: /etc/apache2/sites-enabled/{{ log_prefix }}.conf
  become: true
  when: ansible_facts['os_family'] == "Debian"
```

**Outros ajustes encontrados durante a investigação**

- Removido o `notify` para `InstallWordPress`, um handler que não existia em nenhuma role.
- Corrigida a variável `site_url`, que continha formatação Markdown copiada por engano.
- Ajustada `php_version` para a versão do PHP-FPM instalada no Ubuntu, conferida com `ls /var/run/php/`.

## Lições

- **Handlers só rodam quando a tarefa muda algo.** Um `notify` para um handler inexistente pode passar despercebido por várias execuções e só falhar quando o arquivo de origem é alterado.
- **Distribuições diferentes têm padrões diferentes.** O mesmo pacote (Apache) se comporta de forma distinta no Ubuntu e no Rocky, e a automação precisa tratar isso explicitamente.
- **Corrigir na automação, não no servidor.** Ajustar o Ubuntu à mão resolveria naquele momento, mas a próxima execução ou um servidor novo teriam o mesmo problema.
