
# Introdução aos comandos ad hoc

Um comando ad hoc do Ansible usa a ferramenta de linha de comando `/usr/bin/ansible` para automatizar uma única tarefa em um ou mais nós gerenciados. Comandos ad hoc são rápidos e fáceis, mas não são reutilizáveis. Então, por que aprender sobre comandos ad hoc? Comandos ad hoc demonstram a simplicidade e o poder do Ansible. Os conceitos que você aprende aqui serão transferidos diretamente para a linguagem de playbook. Antes de ler e executar estes exemplos, por favor, leia [Como criar seu inventário](#).

*   Por que usar comandos ad hoc?
*   Casos de uso para tarefas ad hoc
    *   Reiniciar servidores
    *   Gerenciar arquivos
    *   Gerenciar pacotes
    *   Gerenciar usuários e grupos
    *   Gerenciar serviços
    *   Coletar facts
    *   Modo de verificação (Check mode)
    *   Padrões e comandos ad-hoc

## Por que usar comandos ad hoc?

Comandos ad hoc são ótimos para tarefas que você repete raramente. Por exemplo, se você quiser desligar todas as máquinas no seu laboratório para as férias de Natal, você poderia executar um comando de uma linha rápido no Ansible sem escrever um playbook. Um comando ad hoc se parece com isto:

```bash
$ ansible [pattern] -m [module] -a "[module options]"
```

A opção `-a` aceita opções seja através da sintaxe `chave=valor` ou uma string JSON começando com `{` e terminando com `}` para uma estrutura de opções mais complexa. Você pode aprender mais sobre padrões e módulos em outras páginas.

## Casos de uso para tarefas ad hoc

Tarefas ad hoc podem ser usadas para reiniciar servidores, copiar arquivos, gerenciar pacotes e usuários, e muito mais. Você pode usar qualquer módulo do Ansible em uma tarefa ad hoc. Tarefas ad hoc, como playbooks, usam um modelo declarativo, calculando e executando as ações necessárias para alcançar um estado final especificado. Elas alcançam uma forma de idempotência verificando o estado atual antes de começarem e não fazendo nada a menos que o estado atual seja diferente do estado final especificado.

## Reiniciando servidores

O módulo padrão para o utilitário de linha de comando `ansible` é o módulo `ansible.builtin.command`. Você pode usar uma tarefa ad hoc para chamar o módulo command e reiniciar todos os servidores web em Atlanta, 10 por vez. Antes que o Ansible possa fazer isso, você deve ter todos os servidores em Atlanta listados em um grupo chamado `[atlanta]` no seu inventário, e você deve ter credenciais SSH funcionais para cada máquina nesse grupo. Para reiniciar todos os servidores no grupo `[atlanta]`:

```bash
$ ansible atlanta -a "/sbin/reboot"
```

Por padrão, o Ansible usa apenas cinco processos simultâneos. Se você tiver mais hosts do que o valor definido para a contagem de forks, isso pode aumentar o tempo que o Ansible leva para se comunicar com os hosts. Para reiniciar os servidores `[atlanta]` com 10 forks paralelos:

```bash
$ ansible atlanta -a "/sbin/reboot" -f 10
```

O `/usr/bin/ansible` será executado por padrão a partir da sua conta de usuário. Para conectar como um usuário diferente:

```bash
$ ansible atlanta -a "/sbin/reboot" -f 10 -u username
```

Reiniciar provavelmente requer elevação de privilégio. Você pode conectar ao servidor como `username` e executar o comando como o usuário root usando a palavra-chave `become`:

```bash
$ ansible atlanta -a "/sbin/reboot" -f 10 -u username --become [--ask-become-pass]
```

Se você adicionar `--ask-become-pass` ou `-K`, o Ansible solicitará a senha para usar para elevação de privilégio (sudo/su/pfexec/doas/etc).

> **Nota**
>
> O módulo command não suporta sintaxes shell estendidas como pipe e redirecionamentos (embora variáveis de shell sempre funcionem). Se o seu comando requer sintaxe específica do shell, use o módulo shell instead.

Até agora, todos os nossos exemplos usaram o módulo 'command' padrão. Para usar um módulo diferente, passe `-m` para o nome do módulo. Por exemplo, para usar o módulo `ansible.builtin.shell`:

```bash
$ ansible raleigh -m ansible.builtin.shell -a 'echo $TERM'
```

Ao executar qualquer comando com a CLI ad hoc do Ansible (ao contrário de Playbooks), preste atenção especial às regras de aspas do shell, para que o shell local retenha a variável e a passe para o Ansible. Por exemplo, usar aspas duplas em vez de aspas simples no exemplo acima avaliaria a variável na máquina em que você estava.

## Gerenciando arquivos

Uma tarefa ad hoc pode aproveitar o poder do Ansible e SCP para transferir muitos arquivos para múltiplas máquinas em paralelo. Para transferir um arquivo diretamente para todos os servidores no grupo `[atlanta]`:

```bash
$ ansible atlanta -m ansible.builtin.copy -a "src=/etc/hosts dest=/tmp/hosts"
```

Se você planeja repetir uma tarefa como esta, use o módulo `ansible.builtin.template` em um playbook.

O módulo `ansible.builtin.file` permite alterar propriedade e permissões em arquivos. Estas mesmas opções podem ser passadas diretamente ao módulo copy também:

```bash
$ ansible webservers -m ansible.builtin.file -a "dest=/srv/foo/a.txt mode=600"
$ ansible webservers -m ansible.builtin.file -a "dest=/srv/foo/b.txt mode=600 owner=mdehaan group=mdehaan"
```

O módulo file também pode criar diretórios, similar ao `mkdir -p`:

```bash
$ ansible webservers -m ansible.builtin.file -a "dest=/path/to/c mode=755 owner=mdehaan group=mdehaan state=directory"
```

Bem como excluir diretórios (recursivamente) e excluir arquivos:

```bash
$ ansible webservers -m ansible.builtin.file -a "dest=/path/to/c state=absent"
```

## Gerenciando pacotes

Você também pode usar uma tarefa ad hoc para instalar, atualizar ou remover pacotes em nós gerenciados usando um módulo de gerenciamento de pacotes como o yum. Módulos de gerenciamento de pacotes suportam funções comuns para instalar, remover e geralmente gerenciar pacotes. Algumas funções específicas para um gerenciador de pacotes podem não estar presentes no módulo do Ansible já que não fazem parte do gerenciamento geral de pacotes.

Para garantir que um pacote esteja instalado sem atualizá-lo:

```bash
$ ansible webservers -m ansible.builtin.yum -a "name=acme state=present"
```

Para garantir que uma versão específica de um pacote esteja instalada:

```bash
$ ansible webservers -m ansible.builtin.yum -a "name=acme-1.5 state=present"
```

Para garantir que um pacote esteja na versão mais recente:

```bash
$ ansible webservers -m ansible.builtin.yum -a "name=acme state=latest"
```

Para garantir que um pacote não esteja instalado:

```bash
$ ansible webservers -m ansible.builtin.yum -a "name=acme state=absent"
```

O Ansible possui módulos para gerenciar pacotes sob muitas plataformas. Se não houver um módulo para o seu gerenciador de pacotes, você pode instalar pacotes usando o módulo command ou criar um módulo para o seu gerenciador de pacotes.

## Gerenciando usuários e grupos

Você pode criar, gerenciar e remover contas de usuário em seus nós gerenciados com tarefas ad hoc:

```bash
$ ansible all -m ansible.builtin.user -a "name=foo password=<encrypted password here>"
```

```bash
$ ansible all -m ansible.builtin.user -a "name=foo state=absent"
```

Veja a documentação do módulo `ansible.builtin.user` para detalhes sobre todas as opções disponíveis, incluindo como manipular grupos e associação de grupos.

## Gerenciando serviços

Garanta que um serviço esteja iniciado em todos os webservers:

```bash
$ ansible webservers -m ansible.builtin.service -a "name=httpd state=started"
```

Alternativamente, reinicie um serviço em todos os webservers:

```bash
$ ansible webservers -m ansible.builtin.service -a "name=httpd state=restarted"
```

Garanta que um serviço esteja parado:

```bash
$ ansible webservers -m ansible.builtin.service -a "name=httpd state=stopped"
```

## Coletando facts

Facts representam variáveis descobertas sobre um sistema. Você pode usar facts para implementar execução condicional de tarefas, mas também apenas para obter informações ad hoc sobre seus sistemas. Para ver todos os facts:

```bash
$ ansible all -m ansible.builtin.setup
```

Você também pode filtrar esta saída para exibir apenas certos facts, veja a documentação do módulo `ansible.builtin.setup` para detalhes.

## Modo de verificação (Check mode)

No modo de verificação, o Ansible não faz nenhuma alteração nos sistemas remotos. O Ansible imprime apenas os comandos. Ele não executa os comandos.

```bash
$ ansible all -m copy -a "content=foo dest=/root/bar.txt" -C
```

Habilitar o modo de verificação (`-C` ou `--check`) no comando acima significa que o Ansible não cria ou atualiza realmente o arquivo `/root/bar.txt` em nenhum sistema remoto.

## Padrões e comandos ad-hoc

Veja a documentação de padrões para detalhes sobre todas as opções disponíveis, incluindo como limitar o uso de padrões em comandos ad-hoc.

Agora que você entende os elementos básicos de execução do Ansible, você está pronto para aprender a automatizar tarefas repetitivas usando Ansible Playbooks.