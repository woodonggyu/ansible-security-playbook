# How to build your inventory

* [Documentation](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#how-to-build-your-inventory)


## Inventory basics: formats, host, and groups

인벤토리는 Ansible 에서 관리할 호스트의 모음을 정의한 것이다. (default. /etc/ansible/hosts)

가장 일반적인 형식은 ini 또는 YAML 이다. `-i` 옵션을 사용해 cli 에서 인벤토리 파일을 명시적으로 지정할 수 있다.


## Default groups 

디폴트 그룹에는 `all`, `ungroup` 두 가지가 있다. 

두 그룹은 group_names 와 같은 그룹 목록에는 보이지 않지만, 암묵적으로 존재한다.

* `all` 그룹은 모든 호스트가 포함된다.
* `ungroup` 그룹은 다른 그룹을 제외한 모든 호스트가 포함된다.


## Adding ranges of hosts

유사한 패턴을 가진 호스트가 있을경우, 각 호스트 명을 나열하지 않고 범위로 추가할 수 있다.

```yaml
app:
  hosts:
    app[01:50].example.com:
db:
  hosts:
    db[01:50:2].example.com:
test:
  hosts:
    test-[a:z].example.com:
```


## Adding variables to inventory

인벤토리의 특정 호스트나 그룹과 관련된 변수 값을 저장할 수 있다. 

권장 방법으로는, `host_vars`, `groups_vars` 디렉토리를 만들어 호스트 변수와 그룹 변수를 정의한다.

참고로 인벤토리 내 변수 간의 우선순위는 호스트 변수가 그룹 변수보다 우선순위가 높다.


### Assigning a variable to one machine: host variables

단일 호스트에 변수를 할당하고, 이를 플레이븍에서 사용 가능하다.

```ini
[atlanta]
host1 http_port=80 maxRequestsPerChild=808
host2 http_port=303 maxRequestsPerChild=909
```

```yaml
app:
  hosts:
    host1:
      http_port: 80
      maxRequestsPerChild: 808
    host2:
      http_port: 303
      maxRequestsPerChild: 909
```


### Assigning a variable to many machines: group variables

그룹의 모든 호스트가 변수 값을 공유하는 경우, 해당 변수를 전체 그룹에 적용할 수 있다.

```ini
[atlanta]
host1
host2

[atlanta:vars]
ntp_server=ntp.atlanta.example.com
proxy=proxy.atlanta.example.com
```

```yaml
atlanta:
  hosts:
    host1:
    host2:
  vars:
    ntp_server: ntp.atlanta.example.com
    proxy: proxy.atlanta.example.com
```


### Inheriting variable values: group variables for groups of groups

`:children (INI)` 또는 `children: (YAML)` 항목을 사용해 그룹의 하위 그룹을 생성할 수 있다.

그리고 앞서 설명한 `:vars (INI)` 또는 `vars: (YAML)` 를 사용해 하위 그룹에 변수를 적용할 수 있다.

```ini
[atlanta]
host1
host2

[raleigh]
host2
host3

[southeast:children]
atlanta
raleigh

[southeast:vars]
some_server=foo.southeast.example.com
halon_system_timeout=30
self_destruct_countdown=60
escape_pods=2

[usa:children]
southeast
northeast
southwest
northwest
```

```yaml
all:
  children:
    usa:
      children:
        southeast:
          children:
            atlanta:
              hosts:
                host1:
                host2:
            raleigh:
              hosts:
                host2:
                host3:
          vars:
            some_server: foo.southeast.example.com
            halon_system_timeout: 30
            self_destruct_countdown: 60
            escape_pods: 2
        northeast:
        northwest:
        southwest:
```

목록 또는 해시 데이터를 저장해야 하거나 호스트 및 그룹 특정 변수를 인벤토리 파일과 별도로 유지하려는 경 호스트 및 그룹 변수 구성을 참조할 수 있다.

추가로, 하위 그룹에서 주의해야 할 몇 가지 속성은 다음과 같다.

* 하위 그룹의 모든 호스트는 자동으로 상위 그룹에 속한다.
* 하위 그룹의 변수는 상위 그룹의 변수보다 우선순위가 높다. (override)
* 그룹에는 여러 부모와 자식이 있을 수 있지만, 순환 관계는 없다.
* 호스트는 여러 그룹에 속할 수 있지만, 여러 그룹의 데이터를 병합하는 호스트 인스턴스는 하나 뿐이다.


### Organizing host and group variables

기본 인벤토리 파일 내 변수를 저장할 수 있지만 별도의 호스트 및 그룹 변수 파일을 사용해 쉽게 구성할 수 있다.

**호스트 및 변수 그룹 파일은 YAML 구문을 사용해야 한다.** 

Ansible 에서는 인벤토리 파일 또는 플레이북 파일과 관련된 경로를 검색하여 호스트 및 그룹 변수 파일을 로드한다.

단일 파일이 너무 커지거나 일부 그룹 변수에서 Ansible Vault 를 사용하려는 경우, 변수를 체계적으로 유지하는데 유용하다.

만약, `/etc/ansible/hosts` 에 위치한 인벤토리 파일이 'raleigh' 와 'webservers' 라는 두 그룹에 속하는 'foosball' 라는 호스트를 포함한 경우, 해당 호스트는 다음 위치에서 YAML 파일의 변수를 사용한다.
```shell
/etc/ansible/group_vars/raleigh
/etc/ansible/group_vars/webservers
/etc/ansible/host_vars/foosball
```

`group_vars/`, `host_vars/` 디렉토리를 플레이북 디렉토리에 추가할 수 있다.

* ansible-playbook 명령은 기본적으로 현재 작업 중인 디렉토리에서 group_vars, host_vars 디렉토리를 찾는다.
* Ansible 명령 (e.g ansible, ansible-console 등) 은 인벤토리 디렉토리 내의 group_vars, host_vars 디렉토리를 찾는다. 
* 다른 명령이 플레이북 디렉토리에서 그룹 및 호스트 변수를 로드하도록 하고싶다면, 명령줄에 --playbook-dir 옵션을 사용한다.
* 플레이북 디렉토리와 인벤토리 디렉토리 모두에서 파일을 로드 시, 플레이북의 디렉토리의 변수가 인벤토리 디렉토리에 설정된 변수보다 우선순위가 높다.
