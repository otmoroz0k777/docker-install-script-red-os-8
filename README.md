# docker-install-script-red-os-8
#!/bin/bash

set -e

echo "====================================="
echo " Docker installation for RED OS 8"
echo "====================================="

Проверка root
if [ "$EUID" -ne 0 ]; then
echo "Запустите скрипт от root"
exit 1
fi

echo "[1/8] Обновление системы..."
dnf update -y

echo "[2/8] Установка зависимостей..."
dnf install -y yum-utils device-mapper-persistent-data lvm2 curl

echo "[3/8] Добавление Docker repository..."
dnf config-manager
--add-repo
https://download.docker.com/linux/centos/docker-ce.repo

echo "[4/8] Установка Docker..."
dnf install -y docker-ce docker-ce-cli containerd.io

echo "[5/8] Настройка daemon.json..."

mkdir -p /etc/docker

cat > /etc/docker/daemon.json <<EOF
{
"live-restore": false
}
EOF

echo "[6/8] Запуск Docker..."

systemctl daemon-reload
systemctl enable docker
systemctl restart docker

echo "[7/8] Настройка firewall..."

if systemctl is-active firewalld >/dev/null 2>&1; then

firewall-cmd --permanent --add-port=2377/tcp
firewall-cmd --permanent --add-port=7946/tcp
firewall-cmd --permanent --add-port=7946/udp
firewall-cmd --permanent --add-port=4789/udp

firewall-cmd --reload

echo "Firewall настроен"
else
echo "Firewalld не используется — пропуск"
fi

echo "[8/8] Проверка Docker..."

docker --version
systemctl status docker --no-pager

echo ""
echo "====================================="
echo " Docker успешно установлен!"
echo "====================================="
echo ""
echo "Для manager:"
echo "docker swarm init --advertise-addr YOUR_IP"
echo ""
echo "Для worker:"
echo "используйте docker swarm join"
