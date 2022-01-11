# ansible-security-playbook

Ansible which provides playbook helpful security team

## How to build your inventory

* [Documentation](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#how-to-build-your-inventory)


### inventory basics: formats, host, and groups

인벤토리는 Ansible 에서 관리할 호스트의 모음을 정의한 것이다. (default. /etc/ansible/hosts)

가장 일반적인 형식은 ini 또는 YAML 이다. `-i` 옵션을 사용해 cli 에서 인벤토리 파일을 명시적으로 지정할 수 있다.


#### Default groups 

디폴트 그룹에는 `all`, `ungroup` 두 가지가 있다. 두 그룹은 group_names 와 같은 그룹 목록에는 보이지 않지만, 암묵적으로 존재한다.

* `all` 그룹은 모든 호스트가 포함된다.
* `ungroup` 그룹은 다른 그룹을 제외한 모든 호스트가 포함된다.


#### Adding ranges of hosts

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


#### Adding variables to inventory

인벤토리의 특정 호스트나 그룹과 관련된 변수 값을 저장할 수 있다. 




