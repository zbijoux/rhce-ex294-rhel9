# RHCE EX294 Practice Lab

A structured study repository for the Red Hat Certified Engineer exam (EX294) — Ansible automation on RHEL 9. The repo mirrors the exact directory layout expected on the exam control node. Each question has its own doc covering the question text, approach, solution, how to run it, verification steps, and common traps.

---

## Lab environment

| Host | Role | IP |
|---|---|---|
| `control.lab.example.com` | Ansible control node | 172.25.250.254 |
| `node1.lab.example.com` | Managed node — group: `dev` | 172.25.250.9 |
| `node2.lab.example.com` | Managed node — group: `test` | 172.25.250.10 |
| `node3.lab.example.com` | Managed node — groups: `prod`, `webservers` | 172.25.250.11 |
| `node4.lab.example.com` | Managed node — groups: `prod`, `webservers` | 172.25.250.12 |
| `node5.lab.example.com` | Managed node — group: `balancers` | 172.25.250.13 |

SSH user: `student`

---

## Repository structure

```
rhce-ex294/
├── ansible.cfg                   # Control node config (inventory, roles, collections, vault)
├── inventory/
│   └── hosts                     # Static inventory with all groups
├── playbooks/                    # One playbook per exam question
│   ├── yum_repo.yml              # Q02
│   ├── packages.yml              # Q03
│   ├── selinux.yml               # Q04 — see comments inside for Option 1 vs Option 2
│   ├── apache.yml                # Q07
│   ├── roles.yml                 # Q08
│   ├── lv.yml                    # Q09
│   ├── hosts.yml                 # Q10 (do not modify)
│   ├── issue.yml                 # Q11
│   ├── webcontent.yml            # Q12
│   ├── hwreport.yml              # Q13
│   ├── users.yml                 # Q15
│   └── cron.yml                  # Q17
├── roles/
│   ├── requirements.yml          # Q06 — Galaxy roles (balancer, phpinfo)
│   └── apache/                   # Q07 — Custom apache role
│       ├── tasks/main.yml
│       ├── templates/index.html.j2
│       ├── handlers/
│       ├── vars/
│       ├── defaults/
│       └── meta/main.yml
├── collections/
│   └── requirements.yml          # Q05 — Collection tarballs from classroom
├── templates/
│   └── hosts.j2                  # Q10 — Jinja2 hosts template
└── docs/
    ├── q01.md  Install and configure Ansible
    ├── q02.md  Create yum repositories
    ├── q03.md  Install packages
    ├── q04.md  Use the SELinux role        ← two options: RPM copy OR collection FQCN
    ├── q05.md  Install a collection
    ├── q06.md  Install roles using Ansible Galaxy
    ├── q07.md  Create and use the apache role
    ├── q08.md  Use roles from Ansible Galaxy
    ├── q09.md  Create and use a logical volume
    ├── q10.md  Generate a hosts file
    ├── q11.md  Modify file content
    ├── q12.md  Create a web content directory
    ├── q13.md  Generate a hardware report
    ├── q14.md  Create a password vault
    ├── q15.md  Create user accounts
    ├── q16.md  Rekey an Ansible vault
    └── q17.md  Configure a cron job
```

---

## Recommended study flow per question

1. **Read the question** — open `docs/qXX.md`, read the Question section only
2. **Attempt it yourself** — write the playbook or config from memory
3. **Check the approach** — read the Approach section for strategy hints
4. **Compare your solution** — read the Solution section
5. **Run it** — use the exact command in the How to run section
6. **Verify** — run the verification commands and check output
7. **Read the traps** — even if you got it right, review Common exam traps

---

## Initial setup on the control node

```bash
# 1. Clone the repo
git clone https://github.com/YOUR_USERNAME/rhce-ex294.git /home/student/ansible
cd /home/student/ansible

# 2. Install packages (Q01)
sudo dnf -y install ansible-automation-platform-common.noarch ansible-navigator

# 3. Install rhel-system-roles RPM (Q04 Option 1)
sudo dnf -y install rhel-system-roles

# 4. Copy SELinux role into working roles dir so ansible-navigator finds it (Q04 Option 1)
cp -r /usr/share/ansible/roles/rhel-system-roles.selinux roles/

# 5. Install Galaxy roles (Q06)
ansible-galaxy install -r roles/requirements.yml

# 6. Install collections (Q05) — also enables Q04 Option 2
ansible-galaxy collection install -r collections/requirements.yml -p ./collections/

# 7. Create the vault password file (Q14)
echo "whenyouwishuponastar" > /home/student/ansible/secret.txt
chmod 600 secret.txt

# 8. Smoke test
ansible all -m ping
```

---

## Running individual questions

| Q | Topic | Command |
|---|---|---|
| Q02 | yum repos | `ansible-navigator run playbooks/yum_repo.yml -m stdout` |
| Q03 | packages | `ansible-navigator run playbooks/packages.yml -m stdout` |
| Q04 | selinux | `ansible-navigator run playbooks/selinux.yml -m stdout` |
| Q07 | apache role | `ansible-navigator run playbooks/apache.yml -m stdout` |
| Q08 | Galaxy roles | `ansible-navigator run playbooks/roles.yml -m stdout` |
| Q09 | logical volume | `ansible-navigator run playbooks/lv.yml -m stdout` |
| Q10 | hosts file | `ansible-navigator run playbooks/hosts.yml -m stdout` |
| Q11 | /etc/issue | `ansible-navigator run playbooks/issue.yml -m stdout` |
| Q12 | web content | `ansible-navigator run playbooks/webcontent.yml -m stdout` |
| Q13 | hw report | `ansible-navigator run playbooks/hwreport.yml -m stdout` |
| Q15 | users | `ansible-navigator run playbooks/users.yml -m stdout` |
| Q17 | cron | `ansible-navigator run playbooks/cron.yml -m stdout` |

> All playbooks use `ansible-navigator run -m stdout`.
> Q04 also works with `ansible-playbook` if using Option 1 (RPM copy).

---

## Q04 SELinux — two options explained

`ansible-navigator` runs inside an Execution Environment (EE) container. It cannot see RPM-installed roles in `/usr/share/ansible/roles` on the host machine. Two ways to fix this:

**Option 1 — Copy the role** (quickest on exam day):
```bash
sudo dnf -y install rhel-system-roles
cp -r /usr/share/ansible/roles/rhel-system-roles.selinux roles/
```
Then use `name: rhel-system-roles.selinux` in the playbook.

**Option 2 — Use the collection** (no copy needed if Q05 already done):
The `redhat-rhel_system_roles` collection installed in Q05 includes the selinux role.
Use `name: redhat.rhel_system_roles.selinux` in the playbook.

Both run with `ansible-navigator run playbooks/selinux.yml -m stdout`.

---

## Key exam rules to remember

- **`ansible-navigator run -m stdout`** for all playbooks — the `-m stdout` flag gives clean output
- **Never mount the LV** in Q09 — explicitly forbidden
- **Never modify `hosts.yml`** in Q10 — only edit `templates/hosts.j2`
- **Passwords must use `password_hash('sha512')`** in Q15
- **`vault_password_file`** is set in `ansible.cfg` — no extra flags needed for vault operations

---

## Files created during exam (not committed to repo)

```
secret.txt      # vault password (Q14)
locker.yml      # encrypted vault (Q14)
salaries.yml    # rekeyed vault (Q16)
user_list.yml   # downloaded user list (Q15)
```
