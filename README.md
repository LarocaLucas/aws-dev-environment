# ☁️ AWS Cloud Dev Environment

> Ambiente de desenvolvimento completo na nuvem — Ubuntu 24.04 com desktop gráfico acessível via browser, provisionado do zero na AWS.

---

## 📐 Arquitetura

```
┌─────────────────────────────────────────────────────┐
│                     AWS Cloud                        │
│                                                      │
│  ┌─────────────────────────────────────────────┐    │
│  │         EC2 — m7i-flex.large                │    │
│  │         Ubuntu 24.04 LTS                    │    │
│  │         Região: sa-east-1 (São Paulo)        │    │
│  │                                             │    │
│  │  ┌──────────┐   ┌──────────┐               │    │
│  │  │TigerVNC  │──▶│ noVNC   │──▶ Browser     │    │
│  │  │:5901     │   │:6080    │   (porta 6080)  │    │
│  │  └──────────┘   └──────────┘               │    │
│  │                                             │    │
│  │  Elastic IP: 18.229.232.60                  │    │
│  └─────────────────────────────────────────────┘    │
│                                                      │
│  ┌──────────────┐   ┌──────────────────────────┐   │
│  │Security Group│   │       EBS Volume          │   │
│  │SSH: 22       │   │       30GB gp3            │   │
│  │noVNC: 6080   │   └──────────────────────────┘   │
│  └──────────────┘                                   │
└─────────────────────────────────────────────────────┘
```

### Especificações da Instância

| Recurso | Configuração |
|---|---|
| Tipo | m7i-flex.large |
| vCPU | 2 |
| RAM | 8 GB |
| Armazenamento | 30 GB gp3 |
| Sistema Operacional | Ubuntu 24.04.3 LTS |
| Região | sa-east-1 (São Paulo) |
| IP Público | Elastic IP (fixo) |

---

## 🛠️ Stack Instalada

| Ferramenta | Versão | Uso |
|---|---|---|
| VS Code | Latest (snap) | Editor de código |
| Node.js | 20.x LTS | Runtime JavaScript |
| Python | 3.x | Scripts e automação |
| Docker | Latest | Containerização |
| Git | Latest | Controle de versão |
| Google Chrome | Stable | Browser |
| XFCE4 | 4.18 | Desktop gráfico leve |
| TigerVNC | Latest | Servidor VNC |
| noVNC | Latest | Acesso VNC via browser |

---

## 🔐 Acesso

### Via Browser (noVNC)
```
http://18.229.232.60:6080/vnc.html
```

### Via SSH
```bash
ssh -i chave-vm-dev.pem ubuntu@18.229.232.60
```

### Segurança
- Acesso SSH restrito por IP (Security Group)
- Acesso noVNC restrito por IP (Security Group)
- Autenticação VNC por senha
- IMDSv2 habilitado

---

## 📜 Scripts de Setup

### 1. Instalação do Desktop Gráfico

```bash
# Atualizar sistema
sudo apt update && sudo apt upgrade -y

# Instalar XFCE (desktop leve) + TigerVNC + noVNC
sudo apt install -y xfce4 xfce4-goodies \
  tigervnc-standalone-server tigervnc-common \
  novnc websockify xterm x11-xserver-utils dbus-x11

# Configurar senha VNC
vncpasswd

# Configurar xstartup
mkdir -p ~/.vnc
cat > ~/.vnc/xstartup << 'EOF'
#!/bin/bash
export DISPLAY=:1
export XDG_SESSION_TYPE=x11
unset SESSION_MANAGER
unset DBUS_SESSION_BUS_ADDRESS
exec startxfce4
EOF
chmod +x ~/.vnc/xstartup
```

### 2. Serviço VNC (systemd)

```ini
# /etc/systemd/system/vncserver.service
[Unit]
Description=TigerVNC Server
After=network.target

[Service]
Type=forking
User=ubuntu
ExecStartPre=-/usr/bin/vncserver -kill :1
ExecStart=/usr/bin/vncserver :1 -geometry 1920x1080 -depth 24
ExecStop=/usr/bin/vncserver -kill :1
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

### 3. Serviço noVNC (systemd)

```ini
# /etc/systemd/system/novnc.service
[Unit]
Description=noVNC Service
After=vncserver.service

[Service]
Type=simple
User=ubuntu
ExecStart=/usr/bin/websockify --web=/usr/share/novnc/ 6080 localhost:5901
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

### 4. Ativar serviços

```bash
sudo systemctl daemon-reload
sudo systemctl enable vncserver novnc
sudo systemctl start vncserver novnc
```

### 5. Ferramentas de Desenvolvimento

```bash
# Git, curl, wget
sudo apt install -y git curl wget

# Node.js 20
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# Python 3
sudo apt install -y python3 python3-pip python3-venv

# Docker
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER

# VS Code
sudo snap install code --classic

# Google Chrome
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo apt install -y ./google-chrome-stable_current_amd64.deb
```

---

## 💰 Custos

| Recurso | Custo/hora | Custo/mês (8h/dia) |
|---|---|---|
| m7i-flex.large | ~$0.156 | ~$37 |
| EBS 30GB gp3 | — | ~$2.70 |
| Elastic IP (rodando) | Grátis | Grátis |
| **Total estimado** | | **~$40/mês** |

> **Dica:** Interrompa a instância quando não estiver usando para economizar. O Elastic IP só cobra quando a instância está parada.

---

## 🚀 Como Usar

**Iniciar a VM:**
1. Console AWS → EC2 → `vm-dev` → **Iniciar instância**
2. Aguardar ~1 minuto
3. Acessar `http://18.229.232.60:6080/vnc.html`

**Parar a VM:**
1. Console AWS → EC2 → `vm-dev` → **Interromper instância**

**Conectar via SSH:**
```bash
ssh -i chave-vm-dev.pem ubuntu@18.229.232.60
```

---

## 📚 Conceitos AWS Aplicados

- **EC2** — Provisionamento de instância com tipo, AMI e armazenamento
- **Security Groups** — Firewall com regras de entrada por IP e porta
- **Elastic IP** — IP público fixo associado à instância
- **EBS** — Volume de armazenamento persistente gp3
- **IAM** — Boas práticas de segurança com MFA
- **AWS Free Tier / Credits** — Uso dos $200 em créditos para novos usuários
- **Regiões e AZs** — Deploy na região sa-east-1 (São Paulo) para menor latência

---

## 👨‍💻 Autor

**Lucas Laroca Campos**  
Desenvolvedor em formação | Castro, PR, Brasil  
[GitHub](https://github.com/larocalucas) • [LinkedIn](#)
