# Nginx Ansible Role (test_ansible)

## Role overview
Bu Ansible role Nginx serverini ishlab chiqarish sharoitida o‘rnatadi va sozlaydi. U quyidagilarni bajaradi:
- Nginx paketini o‘rnatish
- /etc/nginx konfiguratsiyasini tozalash va qayta yozish
- sayt konfiguratsiyalarini `sites-enabled` papkasiga joylash
- log formatlarini boshqarish
- logrotate konfiguratsiyasini o‘rnatish
- systemd va noyob fayl chegaralarini sozlash
- Nginx xizmatini ishonchli qayta yuklash

## Requirements
- ansible-core 2.15 yoki undan yuqori
- `ansible.posix` kolleksiyasi
- Ubuntu yoki Debian asosli Linux hostlar

## How to run
```bash
ansible-playbook -i hosts.yml playbook.yml
```

## How to add a new site
`group_vars/all.yml` ichida `nginx_sites` ro‘yxatiga yangi sayt ob’ekti qo‘shing.

## How to update allowlist IPs
`group_vars/all.yml` ichida `nginx_allowlist_ips` ro‘yxatini yangilang.

## Verification
```bash
ansible-playbook -i hosts.yml playbook.yml
ansible -i hosts.yml all -m ansible.builtin.shell -a 'nginx -t'
ansible -i hosts.yml all -m ansible.builtin.shell -a 'systemctl status nginx'
```
