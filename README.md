# ansible-class

Automação de **provisionamento e configuração** com **Ansible**, organizada por _roles_ (common, docker, golang) e inventários por ambiente.

## Visão geral
- **Playbook principal:** `playbook.yml`
- **Inventários:** em `environments/<ambiente>/hosts` (ex.: `environments/servidores/hosts`)
- **Grupos sugeridos:** `programming_languages`, `webservers` e `local`
- **Ansible config:** `ansible.cfg` já com `become: true`, _fact caching_ (json), SSH otimizado e `remote_user=ubuntu`.

## Estrutura (resumo)
```
ansible.cfg
environments/
  └─ servidores/
     └─ hosts
playbook.yml
requirements.txt
roles/
  ├─ common/
  ├─ docker/
  └─ golang/
```

### Roles
- **common**: pacotes básicos, timezone, locales, sysctl (ex.: `net.ipv4.ip_forward`), ajustes por versão Ubuntu (20.04/22.04/24.04) e _harden_ de SSH via *drop-in* (`/etc/ssh/sshd_config.d/99-override.conf`).
- **docker**: instala Docker (Debian/Ubuntu e RHEL-like), gera `daemon.json`, opcional *proxy* (`HTTP(S)_PROXY`), habilita serviço e adiciona usuários ao grupo `docker`.
- **golang**: baixa e instala Go (`go_version`), exporta PATH e linka `/usr/local/bin/go`.

## Pré‑requisitos
- **Python 3** no controlador
- Dependências Python (opcionais) listadas em `requirements.txt`:
  ```bash
  pip install -r requirements.txt
  ```
- **Chave SSH** com acesso às máquinas-alvo. Ajuste `ansible_ssh_private_key_file` no inventário ou via `--private-key`.

## Inventário de exemplo
Arquivo `environments/servidores/hosts`:
```ini
[aws]
vini-serve-1 ansible_host=44.223.209.36
vini-serve-2 ansible_host=1.1.1.2
vini-serve-3 ansible_host=1.1.1.3
vini-serve-4 ansible_host=1.1.1.4

[local]
sua-maquina ansible_connection=local

[programming_languages]
vini-serve-1
vini-serve-3

[webservers]
vini-serve-2
vini-serve-4

[aws:vars]
ansible_ssh_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

> Dica: use `group_vars/` e `host_vars/` para variáveis por grupo/host. Evite commitar `.vault_pass.txt` e segredos.

## Execução
```bash
# Todos os hosts, apenas roles 'common' e 'docker'
ansible-playbook -i environments/servidores/hosts playbook.yml --tags common,docker

# Apenas grupo de linguagens, rodando 'golang' e 'nodejs'
ansible-playbook -i environments/servidores/hosts playbook.yml -l programming_languages --tags golang,nodejs

# Apenas webservers, role nginx
ansible-playbook -i environments/servidores/hosts playbook.yml -l webservers --tags nginx
```

### Variáveis úteis (exemplos)
```yaml
# roles/docker/defaults/main.yml
docker_package_state: present
docker_users: ["ubuntu"]
docker_daemon:
  exec-opts: ["native.cgroupdriver=systemd"]
  log-driver: json-file
  log-opts:
    max-size: 10m
    max-file: "3"
  storage-driver: overlay2

# roles/golang/defaults/main.yml
go_version: "1.25.1"
```

## Logs e _fact cache_
- Logs em `logs/ansible/ansible.log`
- _Fact cache_ local em `logs/ansible` (JSON). Pode habilitar Redis se preferir.

## Boas práticas
- Rodar com `--check` e `--diff` em ambientes críticos.
- Usar `ansible-vault` para segredos (não commitar `.vault_pass.txt`).
- Separar inventários por ambiente (`dev`, `stage`, `prod`).