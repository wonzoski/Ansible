# Criando um inventário

Inventários organizam nós gerenciados em arquivos centralizados que fornecem ao Ansible informações do sistema e localizações de rede. Usando um arquivo de inventário, o Ansible pode gerenciar um grande número de hosts com um único comando.

Para completar as seguintes etapas, você precisará do endereço IP ou do nome de domínio totalmente qualificado (FQDN) de pelo menos um sistema host. Para fins de demonstração, o host pode estar rodando localmente em um contêiner ou em uma máquina virtual. Você também deve garantir que sua chave pública SSH seja adicionada ao arquivo `authorized_keys` em cada host.

Continue os primeiros passos com o Ansible e crie um inventário da seguinte forma:

1.  Crie um arquivo chamado `inventory.ini` no diretório `ansible_quickstart` que você criou na etapa anterior.

2.  Adicione um novo grupo `[myhosts]` ao arquivo `inventory.ini` e especifique o endereço IP ou o nome de domínio totalmente qualificado (FQDN) de cada sistema host.

    ```ini
    [myhosts]
    192.0.2.50
    192.0.2.51
    192.0.2.52
    ```

3.  Verifique seu inventário.

    ```bash
    ansible-inventory -i inventory.ini --list
    ```

4.  Faça ping no grupo `myhosts` no seu inventário.

    ```bash
    ansible myhosts -m ping -i inventory.ini
    ```

> **Nota**
>
> Passe a opção `-u` com o comando `ansible` se o nome de usuário for diferente no nó de controle e no(s) nó(s) gerenciado(s).

```json
192.0.2.50 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
192.0.2.51 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
192.0.2.52 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```


Parabéns, você criou um inventário com sucesso. Continue os primeiros passos com o Ansible criando um playbook.

## Inventários no formato INI ou YAML

Você pode criar inventários tanto em arquivos INI quanto em YAML. Na maioria dos casos, como no exemplo das etapas anteriores, arquivos INI são diretos e fáceis de ler para um pequeno número de nós gerenciados.

Criar um inventário no formato YAML torna-se uma opção sensata à medida que o número de nós gerenciados aumenta. Por exemplo, o seguinte é um equivalente do `inventory.ini` que declara nomes únicos para nós gerenciados e usa o campo `ansible_host`:

```yaml
myhosts:
  hosts:
    my_host_01:
      ansible_host: 192.0.2.50
    my_host_02:
      ansible_host: 192.0.2.51
    my_host_03:
      ansible_host: 192.0.2.52
```

## Dicas para criar inventários

*   Certifique-se de que os nomes dos grupos sejam significativos e únicos. Os nomes dos grupos também diferenciam maiúsculas de minúsculas (case sensitive).
*   Evite espaços, hífens e números no início (use `floor_19`, não `19th_floor`) nos nomes dos grupos.
*   Agrupe hosts no seu inventário logicamente de acordo com o **O quê**, **Onde** e **Quando**.
    *   **O quê**
        *   Agrupe hosts de acordo com a topologia, por exemplo: `db`, `web`, `leaf`, `spine`.
    *   **Onde**
        *   Agrupe hosts por localização geográfica, por exemplo: `datacenter`, `region`, `floor`, `building`.
    *   **Quando**
        *   Agrupe hosts por estágio, por exemplo: `development`, `test`, `staging`, `production`.

## Use metagrupos

Crie um metagrupo que organize múltiplos grupos no seu inventário com a seguinte sintaxe:

```yaml
metagroupname:
  children:
```

O seguinte inventário ilustra uma estrutura básica para um data center. Este exemplo de inventário contém um metagrupo `network` que inclui todos os dispositivos de rede e um metagrupo `datacenter` que inclui o grupo `network` e todos os webservers.

```yaml
leafs:
  hosts:
    leaf01:
      ansible_host: 192.0.2.100
    leaf02:
      ansible_host: 192.0.2.110

spines:
  hosts:
    spine01:
      ansible_host: 192.0.2.120
    spine02:
      ansible_host: 192.0.2.130

network:
  children:
    leafs:
    spines:

webservers:
  hosts:
    webserver01:
      ansible_host: 192.0.2.140
    webserver02:
      ansible_host: 192.0.2.150

datacenter:
  children:
    network:
    webservers:
```

## Criar variáveis

Variáveis definem valores para nós gerenciados, como o endereço IP, FQDN, sistema operacional e usuário SSH, para que você não precise passá-los ao executar comandos do Ansible.

Variáveis podem se aplicar a hosts específicos.

```yaml
webservers:
  hosts:
    webserver01:
      ansible_host: 192.0.2.140
      http_port: 80
    webserver02:
      ansible_host: 192.0.2.150
      http_port: 443
```

Variáveis também podem se aplicar a todos os hosts em um grupo.

```yaml
webservers:
  hosts:
    webserver01:
      ansible_host: 192.0.2.140
      http_port: 80
    webserver02:
      ansible_host: 192.0.2.150
      http_port: 443
  vars:
    ansible_user: my_server_user
```