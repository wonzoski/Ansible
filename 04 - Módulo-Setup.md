# Módulo ansible.builtin.setup – Coleta facts sobre hosts remotos

> **Nota**
>
> Este módulo faz parte do ansible-core e está incluído em todas as instalações do Ansible. Na maioria dos casos, você pode usar o nome curto do módulo `setup` mesmo sem especificar a palavra-chave `collections`. No entanto, recomendamos que você use o Nome de Coleção Totalmente Qualificado (FQCN) `ansible.builtin.setup` para facilitar o link para a documentação do módulo e evitar conflitos com outras coleções que possam ter o mesmo nome de módulo.

*   [Sinopse](#sinopse)
*   [Parâmetros](#parâmetros)
*   [Atributos](#atributos)
*   [Notas](#notas)
*   [Exemplos](#exemplos)

## Sinopse

*   Este módulo é chamado automaticamente por playbooks para coletar variáveis úteis sobre hosts remotos que podem ser usadas em playbooks. Ele também pode ser executado diretamente por `/usr/bin/ansible` para verificar quais variáveis estão disponíveis para um host. O Ansible fornece muitos facts sobre o sistema automaticamente.
*   Este módulo também é suportado para alvos Windows.

## Parâmetros

| Parâmetro | Comentários |
|-----------|-------------|
| **fact_path**<br><small>path</small> | Caminho usado para facts locais do ansible (`*.fact`) – arquivos neste diretório serão executados (se forem executáveis) e seus resultados serão adicionados aos facts `ansible_local`. Se um arquivo não for executável, ele será lido. O formato do arquivo/resultados pode ser JSON ou formato INI. O `fact_path` padrão pode ser especificado em `ansible.cfg` para quando o setup for chamado automaticamente como parte do `gather_facts`. **NOTA** – Para clientes Windows, os resultados serão adicionados a uma variável com o nome do arquivo local (sem o sufixo da extensão), em vez de `ansible_local`.<br><br>Desde o Ansible 2.1, hosts Windows podem usar `fact_path`. Certifique-se de que este caminho exista no host de destino. Arquivos neste caminho DEVEM ser scripts PowerShell `.ps1` que produzem um objeto. Este objeto será formatado pelo Ansible como JSON, portanto, o script deve produzir um hashtable bruto, array ou outro objeto primitivo.<br><br>**Padrão:** `"/etc/ansible/facts.d"` |
| **filter**<br><small>lista / elementos=string</small> | Se fornecido, retorna apenas facts que correspondem a um dos padrões no estilo shell (`fnmatch`). Uma lista vazia basicamente significa "sem filtro". A partir do Ansible 2.11, o tipo mudou de string para lista e o padrão se tornou uma lista vazia. Uma string simples ainda é aceita e funciona como um único padrão. O comportamento anterior ao Ansible 2.11 permanece.<br><br>**Padrão:** `[]` |
| **gather_subset**<br><small>lista / elementos=string</small> | Se fornecido, restringe os facts adicionais coletados ao subconjunto especificado. Valores possíveis: `all`, `all_ipv4_addresses`, `all_ipv6_addresses`, `apparmor`, `architecture`, `caps`, `chroot`, `cmdline`, `date_time`, `default_ipv4`, `default_ipv6`, `devices`, `distribution`, `distribution_major_version`, `distribution_release`, `distribution_version`, `dns`, `effective_group_ids`, `effective_user_id`, `env`, `facter`, `fips`, `hardware`, `interfaces`, `is_chroot`, `iscsi`, `kernel`, `local`, `lsb`, `machine`, `machine_id`, `mounts`, `network`, `ohai`, `os_family`, `pkg_mgr`, `platform`, `processor`, `processor_cores`, `processor_count`, `python`, `python_version`, `real_user_id`, `selinux`, `service_mgr`, `ssh_host_key_dsa_public`, `ssh_host_key_ecdsa_public`, `ssh_host_key_ed25519_public`, `ssh_host_key_rsa_public`, `ssh_host_pub_keys`, `ssh_pub_keys`, `system`, `system_capabilities`, `system_capabilities_enforced`, `systemd`, `user`, `user_dir`, `user_gecos`, `user_gid`, `user_id`, `user_shell`, `user_uid`, `virtual`, `virtualization_role`, `virtualization_type`. É possível especificar uma lista de valores para definir um subconjunto maior. Valores também podem ser usados com um `!` inicial para especificar que esse subconjunto específico não deve ser coletado. Por exemplo: `!hardware,!network,!virtual,!ohai,!facter`. Se `!all` for especificado, apenas o subconjunto mínimo é coletado. Para evitar coletar até mesmo o subconjunto mínimo, especifique `!all,!min`. Para coletar apenas facts específicos, use `!all,!min` e especifique os subconjuntos de facts particulares. Use o parâmetro `filter` se não quiser exibir alguns facts coletados.<br><br>**Padrão:** `["all"]` |
| **gather_timeout**<br><small>inteiro</small> | Define o tempo limite padrão em segundos para a coleta individual de facts.<br><br>**Padrão:** `10` |

## Atributos

| Atributo | Suporte | Descrição |
|----------|---------|-----------|
| **check_mode** | full | Pode ser executado em modo de verificação e retornar a previsão de status alterado sem modificar o alvo; se não for suportado, a ação será ignorada. |
| **diff_mode** | none | Retornará detalhes sobre o que mudou (ou possivelmente precisa mudar em `check_mode`), quando estiver em modo diff. |
| **facts** | full | A ação retorna um dicionário `ansible_facts` que atualizará os facts existentes do host. |
| **platform** | Plataformas: posix, windows | Famílias/SOs de destino que podem ser operados. |

## Notas

> **Nota**
>
> *   Mais facts do ansible serão adicionados com versões sucessivas. Se o `facter` ou `ohai` estiverem instalados, variáveis desses programas também serão capturadas no arquivo JSON para uso em templating. Essas variáveis são prefixadas com `facter_` e `ohai_`, então é fácil identificar sua origem. Todas as variáveis são propagadas para o chamador. Usar os facts do ansible e optar por não instalar o facter e ohai significa que você pode evitar dependências do Ruby em seus sistemas remotos. (Veja também `community.general.facter` e `community.general.ohai`.)
> *   A opção `filter` filtra apenas o subchave de primeiro nível abaixo de `ansible_facts`.
> *   Se o host de destino for Windows, você não terá atualmente a capacidade de usar `filter`, pois isso é fornecido por uma implementação mais simples do módulo.
> *   Este módulo deve ser executado com privilégios elevados em sistemas BSD para coletar facts como `ansible_product_version`.
> *   Para mais informações sobre facts delegados, verifique https://docs.ansible.com/ansible/latest/user_guide/playbooks_delegation.html#delegating-facts.

## Exemplos

```bash
# Exibe facts de todos os hosts e os armazena indexados por `hostname` em `/tmp/facts`.
# ansible all -m ansible.builtin.setup --tree /tmp/facts

# Exibe apenas facts relacionados à memória encontrados pelo ansible em todos os hosts e os exibe.
# ansible all -m ansible.builtin.setup -a 'filter=ansible_*_mb'

# Exibe apenas facts retornados pelo facter.
# ansible all -m ansible.builtin.setup -a 'filter=facter_*'

# Coleta apenas facts retornados pelo facter.
# ansible all -m ansible.builtin.setup -a 'gather_subset=!all,facter'
```

```yaml
- name: Coleta apenas facts retornados pelo facter
  ansible.builtin.setup:
    gather_subset:
      - '!all'
      - '!<qualquer subconjunto válido>'
      - facter

- name: Filtra e retorna apenas facts selecionados
  ansible.builtin.setup:
    filter:
      - 'ansible_distribution'
      - 'ansible_machine_id'
      - 'ansible_*_mb'
```

```bash
# Exibe apenas facts sobre determinadas interfaces.
# ansible all -m ansible.builtin.setup -a 'filter=ansible_eth[0-2]'

# Restringe facts adicionais coletados apenas para network e virtual (inclui facts mínimos padrão)
# ansible all -m ansible.builtin.setup -a 'gather_subset=network,virtual'

# Coleta apenas network e virtual (exclui facts mínimos padrão)
# ansible all -m ansible.builtin.setup -a 'gather_subset=!all,network,virtual'

# Não chama o puppet facter ou ohai mesmo se estiverem presentes.
# ansible all -m ansible.builtin.setup -a 'gather_subset=!facter,!ohai'

# Coleta apenas a quantidade mínima padrão de facts:
# ansible all -m ansible.builtin.setup -a 'gather_subset=!all'

# Não coleta facts, nem mesmo o subconjunto mínimo padrão de facts:
# ansible all -m ansible.builtin.setup -a 'gather_subset=!all,!min'

# Exibe facts de hosts Windows com facts personalizados armazenados em C:\custom_facts.
# ansible windows -m ansible.builtin.setup -a "fact_path='c:\custom_facts'"
```

```yaml
# Coleta facts para as máquinas no grupo dbservers (também conhecido como Delegating facts)
- hosts: app_servers
  tasks:
    - name: Coleta facts dos servidores de banco de dados
      ansible.builtin.setup:
      delegate_to: "{{ item }}"
      delegate_facts: true
      loop: "{{ groups['dbservers'] }}"
```

