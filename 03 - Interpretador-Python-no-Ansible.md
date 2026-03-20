# Suporte ao Python 3

O Ansible 2.5 e versões superiores funcionam com Python 3. Antes do 2.5, o uso do Python 3 era considerado uma prévia técnica. Este tópico discute como configurar seu nó de controle e máquinas gerenciadas para usar Python 3.

Consulte **Requisitos do Nó de Controle** e **Requisitos do Nó Gerenciado** para as versões específicas suportadas.

## No lado do nó de controle

A maneira mais fácil de executar `/usr/bin/ansible` sob Python 3 é instalá-lo com a versão Python3 do pip. Isso fará com que o `/usr/bin/ansible` padrão seja executado com Python3:

```bash
$ pip3 install ansible
$ ansible --version | grep "python version"
  python version = 3.10.5 (main, Jun 9 2022, 00:00:00) [GCC 12.1.1 20220507 (Red Hat 12.1.1-1)] (/usr/bin/python)
```

Se você estiver executando o Ansible na branch `devel` a partir de um clone e quiser usar Python 3 com seu checkout de fonte, execute seu comando através do `python3`. Por exemplo:

```bash
$ source ./hacking/env-setup
$ python3 $(which ansible) localhost -m ping
$ python3 $(which ansible-playbook) sample-playbook.yml
```

> **Nota**
>
> Pacotes individuais de distribuição Linux podem ser empacotados para Python2 ou Python3. Ao executar a partir de pacotes da distro, você só poderá usar o Ansible com a versão do Python para a qual ele foi instalado. Às vezes, as distros fornecerão um meio de instalar para várias versões do Python (através de um pacote separado ou através de alguns comandos executados após a instalação). Você precisará verificar com sua distro para ver se isso se aplica ao seu caso.

## Usando Python 3 nas máquinas gerenciadas com comandos e playbooks

O Ansible detectará e usará automaticamente o Python 3 em muitas plataformas que são distribuídas com ele. Para configurar explicitamente um intérprete Python 3, defina a variável de inventário `ansible_python_interpreter` no nível do grupo ou do host para o local de um intérprete Python 3, como `/usr/bin/python3`. O caminho do intérprete padrão também pode ser definido em `ansible.cfg`.

**Veja também**

*   Descoberta de Intérprete para mais informações.

```ini
# Exemplo de inventário que cria um alias para localhost que usa Python3
localhost-py3 ansible_host=localhost ansible_connection=local ansible_python_interpreter=/usr/bin/python3

# Exemplo de configuração de um grupo de hosts para usar Python3
[py3_hosts]
ubuntu16
fedora27

[py3_hosts:vars]
ansible_python_interpreter=/usr/bin/python3
```

**Veja também**

*   Como criar seu inventário para mais informações.

Execute seu comando ou playbook:

```bash
$ ansible localhost-py3 -m ping
$ ansible-playbook sample-playbook.yml
```

Note que você também pode usar a opção de linha de comando `-e` para definir manualmente o intérprete python quando executar um comando. Isso pode ser útil se você quiser testar se um módulo ou playbook específico possui algum bug sob Python 3. Por exemplo:

```bash
$ ansible localhost -m ping -e 'ansible_python_interpreter=/usr/bin/python3'
$ ansible-playbook sample-playbook.yml -e 'ansible_python_interpreter=/usr/bin/python3'
```

## O que fazer se uma incompatibilidade for encontrada

Passamos várias versões corrigindo bugs e adicionando novos testes para que o conjunto de recursos principais do Ansible funcione sob Python 2 e Python 3. No entanto, bugs ainda podem existir em casos extremos e muitos dos módulos distribuídos com o Ansible são mantidos pela comunidade e nem todos podem ter sido portados ainda.

Se você encontrar um bug executando sob Python 3, pode enviar um relatório de bug no projeto do Ansible no GitHub. Certifique-se de mencionar Python3 no relatório de bug para que as pessoas certas o analisem.

Se você gostaria de corrigir o código e enviar um pull request no github, pode consultar **Ansible e Python 3** para obter informações sobre como corrigimos problemas comuns de compatibilidade com Python3 na base de código do Ansible.

## Exemplo de configuração no ansible.cfg
Adicione as seguintes instruções no arquivo ansible.cfg:
```ini
[defaults]
# Sets a specific path
interpreter_python = /usr/bin/python3

# OR use auto-discovery (recommended for mixed environments)
# interpreter_python = auto_silent
```
