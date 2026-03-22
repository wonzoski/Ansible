# Playbooks do Ansible

Os Playbooks do Ansible fornecem um sistema de gerenciamento de configuração e implantação em múltiplas máquinas que é repetível, reutilizável e simples, sendo muito adequado para implantar aplicações complexas. Se você precisar executar uma tarefa com o Ansible mais de uma vez, pode escrever um playbook e colocá-lo sob controle de versão. Você pode então usar o playbook para aplicar novas configurações ou confirmar a configuração de sistemas remotos.

Playbooks permitem que você execute as seguintes ações:

*   Declarar configurações.
*   Orquestrar etapas de qualquer processo manual ordenado em múltiplos conjuntos de máquinas em uma ordem definida.
*   Executar tarefas de forma síncrona ou assíncrona.

*   [Sintaxe do Playbook](#sintaxe-do-playbook)
*   [Execução do Playbook](#execução-do-playbook)
    *   [Execução de tarefas](#execução-de-tarefas)
    *   [Estado desejado e idempotência](#estado-desejado-e-idempotência)
    *   [Executando playbooks](#executando-playbooks)
    *   [Executando playbooks em modo de verificação](#executando-playbooks-em-modo-de-verificação)
*   [Ansible-Pull](#ansible-pull)
*   [Verificando playbooks](#verificando-playbooks)
    *   [ansible-lint](#ansible-lint)

## Sintaxe do Playbook

Você expressa playbooks no formato YAML com uma sintaxe mínima. Se você não estiver familiarizado com YAML, revise a [visão geral da Sintaxe YAML](https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html) e considere instalar um complemento para seu editor de texto (veja [Outras Ferramentas e Programas](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#other-tools-and-programs)) para ajudá-lo a escrever sintaxe YAML limpa em seus playbooks.

Um playbook consiste em um ou mais "plays" em uma lista ordenada. Os termos "playbook" e "play" são analogias esportivas. Cada play executa parte do objetivo geral do playbook, executando uma ou mais tarefas. Cada tarefa chama um módulo do Ansible.

## Execução do Playbook

Um playbook é executado em ordem, de cima para baixo. Dentro de cada play, as tarefas também são executadas em ordem, de cima para baixo. Playbooks com múltiplos plays podem orquestrar implantações em múltiplas máquinas, executando um play em seus servidores web, outro play em seus servidores de banco de dados e um terceiro play em sua infraestrutura de rede. No mínimo, cada play define duas coisas:

*   Os nós gerenciados como alvo, usando um padrão.
*   Pelo menos uma tarefa para executar.

Para Ansible 2.10 e versões posteriores, você deve usar o nome de coleção totalmente qualificado (FQCN) em seus playbooks. Usar o FQCN garante que você selecionou o módulo correto, pois múltiplas coleções podem conter módulos com o mesmo nome. Por exemplo, `user`. Consulte [Usando coleções em um playbook](https://docs.ansible.com/ansible/latest/user_guide/collections_using.html#using-collections-in-a-playbook).

No exemplo a seguir, o primeiro play tem como alvo os servidores web e o segundo play tem como alvo os servidores de banco de dados.

```yaml
---
- name: Atualizar servidores web
  hosts: webservers
  remote_user: root

  tasks:
  - name: Garantir que o apache esteja na versão mais recente
    ansible.builtin.yum:
      name: httpd
      state: latest

  - name: Escrever o arquivo de configuração do apache
    ansible.builtin.template:
      src: /srv/httpd.j2
      dest: /etc/httpd.conf

- name: Atualizar servidores de banco de dados
  hosts: databases
  remote_user: root

  tasks:
  - name: Garantir que o postgresql esteja na versão mais recente
    ansible.builtin.yum:
      name: postgresql
      state: latest

  - name: Garantir que o postgresql esteja iniciado
    ansible.builtin.service:
      name: postgresql
      state: started
```

Seu playbook pode incluir mais do que apenas uma linha `hosts` e `tasks`. Por exemplo, o playbook acima define um `remote_user` para cada play. O `remote_user` é a conta de usuário para a conexão SSH. Você pode adicionar outras [Palavras-chave de Playbook](https://docs.ansible.com/ansible/latest/reference_appendices/playbooks_keywords.html) no nível do playbook, play ou tarefa para influenciar como o Ansible se comporta. Palavras-chave de playbook podem controlar o plugin de conexão, se deve usar elevação de privilégio, como lidar com erros e muito mais. Para suportar uma variedade de ambientes, você pode definir muitos desses parâmetros como flags de linha de comando em sua configuração do Ansible ou em seu inventário. Aprender as regras de precedência para essas fontes de dados ajuda você à medida que expande seu ecossistema Ansible.

### Execução de tarefas

Por padrão, o Ansible executa cada tarefa em ordem, uma de cada vez, contra todas as máquinas correspondentes ao padrão de host. Cada tarefa executa um módulo com argumentos específicos. Após uma tarefa ter sido executada em todas as máquinas de destino, o Ansible passa para a próxima tarefa. Você pode usar estratégias para alterar esse comportamento padrão. Dentro de cada play, o Ansible aplica as mesmas diretivas de tarefa a todos os hosts. Se uma tarefa falhar em um host, o Ansible remove esse host da rotação para o restante do playbook.

Quando você executa um playbook, o Ansible retorna informações sobre conexões, as linhas de nome de todos os seus plays e tarefas, se cada tarefa teve sucesso ou falhou em cada máquina e se cada tarefa fez uma alteração em cada máquina. No final da execução do playbook, o Ansible fornece um resumo dos nós que foram alvo e como eles se saíram. Falhas gerais e tentativas de comunicação fatais "inacessíveis" são mantidas separadas nas contagens.

### Estado desejado e idempotência

A maioria dos módulos do Ansible verifica se o estado final desejado já foi alcançado e sai sem executar nenhuma ação se esse estado tiver sido alcançado. Repetir a tarefa não altera o estado final. Módulos que se comportam dessa forma são "idempotentes". Quer você execute um playbook uma vez ou várias vezes, o resultado deve ser o mesmo. No entanto, nem todos os playbooks e nem todos os módulos se comportam dessa forma. Se não tiver certeza, teste seus playbooks em um ambiente de sandbox antes de executá-los várias vezes em produção.

### Executando playbooks

Para executar seu playbook, use o comando `ansible-playbook`.

```bash
ansible-playbook playbook.yml -f 10
```

Use a flag `--verbose` ao executar seu playbook para ver a saída detalhada de tarefas bem-sucedidas e malsucedidas.

### Executando playbooks em modo de verificação

O modo de verificação do Ansible permite que você execute um playbook sem aplicar nenhuma alteração aos seus sistemas. Você pode usar o modo de verificação para testar playbooks antes de implementá-los em um ambiente de produção.

Para executar um playbook em modo de verificação, passe a flag `-C` ou `--check` para o comando `ansible-playbook`:

```bash
ansible-playbook --check playbook.yaml
```

Executar este comando executa o playbook normalmente. Em vez de implementar quaisquer modificações, o Ansible fornece um relatório sobre as alterações que teria feito. Este relatório inclui detalhes como modificações de arquivos, execução de comandos e chamadas de módulos.

O modo de verificação oferece uma abordagem segura e prática para examinar a funcionalidade de seus playbooks sem correr o risco de alterações não intencionais em seus sistemas. O modo de verificação também é uma ferramenta valiosa para solucionar problemas em playbooks que não estão funcionando como esperado.

## Ansible-Pull

Você pode inverter a arquitetura do Ansible para que os nós façam check-in em um local central em vez de você aplicar a configuração neles.

O comando `ansible-pull` é um pequeno script que faz checkout de um repositório de instruções de configuração do git e então executa `ansible-playbook` contra esse conteúdo.

Se você balancear a carga do seu local de checkout, o `ansible-pull` escala infinitamente.

Execute `ansible-pull --help` para obter detalhes.

## Verificando playbooks

Você pode querer verificar seus playbooks para capturar erros de sintaxe e outros problemas antes de executá-los. O comando `ansible-playbook` oferece várias opções para verificação, incluindo `--check`, `--diff`, `--list-hosts`, `--list-tasks` e `--syntax-check`. O tópico [Ferramentas para validar playbooks](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_checkpoints.html) descreve outras ferramentas para validar e testar playbooks.

### ansible-lint

Você pode usar o `ansible-lint` para obter feedback detalhado e específico do Ansible sobre seus playbooks antes de executá-los. Por exemplo, se você executar `ansible-lint` no playbook chamado `verify-apache.yml` próximo ao topo desta página, você deve obter os seguintes resultados:

```bash
$ ansible-lint verify-apache.yml
[403] Package installs should not use latest
verify-apache.yml:8
Task/Handler: ensure apache is at the latest version
```

A página de regras padrão do `ansible-lint` descreve cada erro.