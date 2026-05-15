# Migracion Linux Mint → Omarchy

Guia completa para migrar OpenCode + Gentle-AI + Engram Sync de Linux Mint a Omarchy (Arch + Hyprland).

## Fase 0: Pre-migracion (en la maquina actual, ANTES de instalar Omarchy)

### 0.1 Verificar que todo este pusheado a GitHub

```bash
# Verificar estado de cada repo
cd ~/dev/personal/opencode-dotfiles && git status
cd ~/dev/personal/engineer-knowledge && git status
cd ~/dev/personal/stack-pilot-agents/stack-pilot && git status
cd ~/.engram-sync && git status
```

Si hay cambios pendientes:
```bash
cd <repo> && git add -A && git commit -m "pre-migration sync" && git push
```

### 0.2 Exportar memorias de Engram

```bash
cd ~/.engram-sync
engram sync --all           # Exportar todas las memorias nuevas
git add . && git commit -m "pre-migration engram sync" && git push
```

### 0.3 Backup de claves SSH

```bash
# Copiar claves SSH a un pendrive o servicio seguro
cp -r ~/.ssh/id_* /path/to/usb/
```

### 0.4 Anotar configuraciones importantes

- **Git config**: `git config --global --list`
- **Keyboard layout**: es-AR (Latin American)
- **Timezone**: America/Argentina/Buenos_Aires (o la que corresponda)
- **Monitores**: ultrawide 3440x1440 + 1920x1080, scaling 1x

---

## Fase 1: Instalar Omarchy

### 1.1 Descargar la ISO

```
https://iso.omarchy.org/omarchy-3.8.0.iso
```

### 1.2 Grabar en pendrive

```bash
# En Linux Mint:
sudo dd if=omarchy-3.8.0.iso of=/dev/sdX bs=4M status=progress

# O usar caligula:
caligula burn omarchy-3.8.0.iso
```

### 1.3 Desactivar Secure Boot

Entrar al BIOS (F2 o Fn+F2 en Yoga Slim 7):
1. Security → Secure Boot → **Disabled**
2. Security → TPM → **Disabled**

### 1.4 Instalar desde el pendrive

1. Bootea desde USB
2. Respondé las preguntas de configuracion (idioma, teclado, timezone, usuario)
3. Selecciona el disco (va a borrar TODO el disco)
4. Espera 2-10 minutos
5. Reinicia

**Importante**: Usar teclado con cable o dongle 2.4GHz para ingresar la passphrase de encriptacion. Bluetooth NO funciona en el unlock.

---

## Fase 2: Configurar Omarchy post-instalacion

### 2.1 Ajustar monitores (ultrawide + 1080p)

Omarchy asume monitores retina por defecto. Para pantallas estandar:

```bash
# Abrir configuracion de monitores
# Super + Alt + Space → Setup → Monitors
# O editar directamente:
nvim ~/.config/hypr/monitors.conf
```

Configuracion recomendada para dual monitor con scaling 1x:

```
env = GDK_SCALE,1
monitor=,preferred,auto,1
```

Para posicionar los monitores:
```bash
# Ver los nombres de los monitores
hyprctl monitors

# Editar la posicion
# Monitor principal (ultrawide) a la izquierda
# Monitor secundario (1080p) a la derecha
```

Tambien podes ciclar el scaling con: `Super + /` (subir) y `Super + Alt + /` (bajar).

### 2.2 Keyboard layout (es-AR)

```bash
# Super + Alt + Space → Setup → Input
# O editar:
nvim ~/.config/hypr/input.conf
```

Agregar:
```
input {
    kb_layout = us,es
    kb_options = grp:alt_shift_toggle
}
```

### 2.3 Actualizar Omarchy

```bash
# Super + Alt + Space → Update → Omarchy
# O desde terminal:
omarchy update
```

### 2.4 Instalar herramientas base

```bash
# Herramientas esenciales (la mayoria ya vienen con Omarchy/Arch)
sudo pacman -S --needed git go nodejs npm

# Herramientas modernas (ya vienen con Omarchy pero por si acaso)
# bat, rg (ripgrep), fd, sd, eza, fzf, zoxide
```

---

## Fase 3: Instalar Engram

### 3.1 Instalar Go (si no esta)

```bash
# Verificar si ya esta instalado (viene con Omarchy)
go version

# Si no esta:
omarchy pkg add go
# O manualmente:
sudo pacman -S go
```

### 3.2 Instalar Engram

```bash
go install github.com/Gentleman-Programming/engram/cmd/engram@latest
```

### 3.3 Verificar

```bash
engram --version
# Deberia mostrar v1.15.x o superior
```

---

## Fase 4: Restaurar memorias de Engram

### 4.1 Clonar el repo de memorias

```bash
git clone https://github.com/TasioDemarchi/engram-memories.git ~/.engram-sync
```

### 4.2 Configurar variable de entorno

```bash
echo '# Engram sync directory' >> ~/.bashrc
echo 'export ENGRAM_DATA_DIR="$HOME/.engram-sync"' >> ~/.bashrc
source ~/.bashrc
```

### 4.3 Importar las memorias

```bash
engram sync --import
engram sync --status
```

Deberia mostrar algo como:
```
Local chunks:    2
Remote chunks:   2
Pending import:  0
```

### 4.4 Configurar Engram como servicio (opcional pero recomendado)

```bash
mkdir -p ~/.config/systemd/user

cat > ~/.config/systemd/user/engram.service << 'EOF'
[Unit]
Description=Engram Memory Server
After=network.target

[Service]
WorkingDirectory=%h
ExecStart=%h/.local/bin/engram serve
Restart=always
RestartSec=3
Environment=ENGRAM_DATA_DIR=%h/.engram-sync

[Install]
WantedBy=default.target
EOF

systemctl --user enable --now engram.service
```

### 4.5 Mantener el sync actualizado

Para sincronizar nuevas memorias antes y despues de cada sesion:

```bash
# Antes de empezar a trabajar (traer cambios del remoto)
cd ~/.engram-sync && git pull && engram sync --import

# Al finalizar (guardar cambios localmente y en el remoto)
engram sync --all && cd ~/.engram-sync && git add . && git commit -m "sync engram memories" && git push
```

---

## Fase 5: Configurar OpenCode + Gentle-AI

### 5.1 Clonar opencode-dotfiles

```bash
mkdir -p ~/dev/personal
cd ~/dev/personal
git clone https://github.com/TasioDemarchi/opencode-dotfiles.git
```

### 5.2 Instalar OpenCode

Omarchy ya viene con OpenCode pre-instalado. Verificar:

```bash
opencode --version
```

Si por alguna razon no esta:
```bash
# Omarchy usa mise para gestionar runtimes
mise use node@latest
npm install -g opencode-ai
```

### 5.3 Copiar configuracion

```bash
# Crear directorio de config
mkdir -p ~/.config/opencode

# Copiar todo el contenido del repo
cp -r ~/dev/personal/opencode-dotfiles/* ~/.config/opencode/

# Copiar archivos ocultos si existen
cp -r ~/dev/personal/opencode-dotfiles/.* ~/.config/opencode/ 2>/dev/null
```

### 5.4 Instalar dependencias de los plugins

Los plugins usan Bun (ya viene con Omarchy):

```bash
cd ~/.config/opencode
bun install
```

### 5.5 Verificar la configuracion

```bash
# Verificar que los paths relativos funcionen
cat ~/.config/opencode/opencode.json | grep "file:" | head -5
# Deberia mostrar: {file:./AGENTS.md}, {file:./prompts/sdd/sdd-apply.md}, etc.

# Verificar que el shell sea bash
cat ~/.config/opencode/opencode.json | grep shell
# Deberia mostrar: "shell": "bash"

# Verificar que engram use el binary correcto
cat ~/.config/opencode/opencode.json | grep -A3 '"engram"'
# Deberia mostrar: "command": ["engram", "mcp", "--tools=agent"]
```

### 5.6 Configurar git

```bash
# Copiar la config de git del repo
cp ~/dev/personal/opencode-dotfiles/.gitconfig ~/.gitconfig

# O configurar manualmente:
git config --global user.name "TasioDemarchi"
git config --global user.email "demarchitasio@gmail.com"
git config --global core.editor nvim
git config --global init.defaultBranch main
```

### 5.7 Probar OpenCode

```bash
cd ~/dev/personal
opencode
```

Dentro de OpenCode, probar:
- `mem_search query: "migracion linux"` → Deberia encontrar las memorias
- `mem_context` → Deberia mostrar las sesiones anteriores

---

## Fase 6: Clonar proyectos

```bash
cd ~/dev/personal

# Proyectos principales
git clone https://github.com/TasioDemarchi/engineer-knowledge.git
git clone https://github.com/TasioDemarchi/stack-pilot.git

# Stack-pilot-agents (si existe)
git clone https://github.com/TasioDemarchi/stack-pilot-agents.git
```

---

## Fase 7: Configuracion adicional para Java (si es necesario)

Si necesitas Java para proyectos:

```bash
# Instalar JDK
sudo pacman -S jdk-openjdk

# O si preferis una version especifica:
sudo pacman -S jdk17-openjdk  # Java 17
sudo pacman -S jdk21-openjdk  # Java 21

# IntelliJ IDEA (opcion via AUR)
omarchy pkg add intellij-idea-community-edition
# O instalar manualmente desde JetBrains Toolbox
```

---

## Fase 8: Verificacion completa

| Check | Comando | Esperado |
|-------|---------|----------|
| OpenCode funciona | `opencode --version` | Version actual |
| Engram funciona | `engram --version` | v1.15.x+ |
| Engram MCP conecta | Abrir OpenCode y usar `mem_search` | Resultados |
| Engram sync funciona | `engram sync --status` | 2+ chunks, 0 pending |
| Git configurado | `git config --global user.name` | TasioDemarchi |
| Proyectos clonados | `ls ~/dev/personal/` | Todos los repos |
| Monitores OK | `hyprctl monitors` | Ambos monitores |
| SSH keys OK | `ssh -T git@github.com` | Hi TasioDemarchi! |

---

## Comandos utiles de Omarchy

| Accion | Comando |
|--------|---------|
| Menu principal | `Super + Alt + Space` |
| Launcher de apps | `Super + Space` |
| Terminal | `Super + Return` |
| Browser | `Super + Shift + Return` |
| Cerrar ventana | `Super + W` |
| Fullscreen | `Super + F` |
| Cambiar layout tiling | `Super + J` |
| Ver atajos | `Super + K` |
| System menu | `Super + Escape` |
| Escalar monitores | `Super + /` / `Super + Alt + /` |
| Temas | `Super + Ctrl + Shift + Space` |
| Actualizar Omarchy | `omarchy update` |
| Instalar paquete | `omarchy pkg add <paquete>` |
| Instalar AUR | `omarchy pkg add <paquete>` (busca en AUR) |
| Configs | `Super + Alt + Space → Setup → Configs` |

---

## Estructura de directorios en Omarchy

```
~/
├── .config/
│   ├── opencode/           # Config de OpenCode (desde opencode-dotfiles)
│   │   ├── opencode.json   # Config principal (paths relativos)
│   │   ├── AGENTS.md       # Instrucciones del agente
│   │   ├── plugins/        # Plugins (engram.ts, background-agents.ts)
│   │   ├── prompts/        # Prompts de agentes
│   │   ├── commands/      # Comandos slash
│   │   ├── skills/         # Skills de SDD
│   │   └── themes/         # Temas
│   └── hypr/               # Config de Hyprland
│       ├── hyprland.conf   # Config principal
│       ├── monitors.conf   # Config de monitores
│       └── input.conf      # Keyboard, mouse, trackpad
├── .engram-sync/           # Repo git de memorias de Engram
│   └── .engram/
│       ├── manifest.json   # Indice de chunks
│       ├── chunks/         # Chunks comprimidos
│       └── engram.db       # DB local (gitignored)
├── .local/
│   ├── bin/engram          # Binario de Engram
│   └── share/opencode/     # Datos de OpenCode
└── dev/personal/
    ├── opencode-dotfiles/  # Repo fuente de la config
    ├── engineer-knowledge/ # Base de conocimiento
    └── stack-pilot-agents/ # Proyectos
```

---

## Troubleshooting

### Engram no conecta en OpenCode

```bash
# Verificar que el servidor de Engram esta corriendo
curl http://127.0.0.1:7437/health

# Si no responde, iniciarlo manualmente:
engram serve &

# Verificar que el binary esta en el PATH
which engram
```

### Los monitores no se detectan

```bash
# Listar monitores conectados
hyprctl monitors

# Re-cargar configuracion
hyprctl reload

# Si un monitor no aparece, forzar:
# Editar ~/.config/hypr/monitors.conf
```

### OpenCode no encuentra los prompts

Los paths en `opencode.json` son relativos (`{file:./prompts/...}`). OpenCode los resuelve desde el directorio donde esta el `opencode.json`. Asegurate de que la estructura sea:

```
~/.config/opencode/
├── opencode.json    # {file:./prompts/...} → busca en ./prompts/
├── prompts/
│   ├── agents/
│   │   └── engineering-sensei.md
│   └── sdd/
│       ├── sdd-apply.md
│       ├── sdd-spec.md
│       └── ...
├── plugins/
│   ├── engram.ts
│   └── background-agents.ts
└── AGENTS.md
```

### SSH no conecta a GitHub

```bash
# Generar nueva clave SSH (si no copiaste la vieja)
ssh-keygen -t ed25519 -C "demarchitasio@gmail.com"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# Agregar a GitHub
cat ~/.ssh/id_ed25519.pub
# Copiar y pegar en https://github.com/settings/keys

# Verificar
ssh -T git@github.com
```

---

## Script de migracion rapida (one-liner)

Si prefieres correr todo de una vez despues de instalar Omarchy:

```bash
# Migracion completa en un solo script
mkdir -p ~/dev/personal && \
git clone https://github.com/TasioDemarchi/engram-memories.git ~/.engram-sync && \
echo 'export ENGRAM_DATA_DIR="$HOME/.engram-sync"' >> ~/.bashrc && \
source ~/.bashrc && \
engram sync --import && \
cd ~/dev/personal && \
git clone https://github.com/TasioDemarchi/opencode-dotfiles.git && \
git clone https://github.com/TasioDemarchi/engineer-knowledge.git && \
git clone https://github.com/TasioDemarchi/stack-pilot.git && \
mkdir -p ~/.config/opencode && \
cp -r ~/dev/personal/opencode-dotfiles/* ~/.config/opencode/ && \
cp -r ~/dev/personal/opencode-dotfiles/.* ~/.config/opencode/ 2>/dev/null && \
cd ~/.config/opencode && bun install && \
go install github.com/Gentleman-Programming/engram/cmd/engram@latest && \
echo "Migracion completa. Ejecuta 'opencode' para verificar."
```

---

## Checklist final

- [ ] Omarchy instalada y booteando
- [ ] Secure Boot desactivado
- [ ] Monitores configurados (1x scaling)
- [ ] Keyboard layout es-AR configurado
- [ ] Engram instalado y `engram --version` OK
- [ ] Engram sync importado y `engram sync --status` OK
- [ ] Engram MCP conecta en OpenCode
- [ ] OpenCode configurado con `opencode-dotfiles`
- [ ] `opencode.json` tiene paths relativos (`./prompts/...`)
- [ ] `opencode.json` tiene `shell: bash`
- [ ] `opencode.json` tiene `engram` con `command: ["engram", "mcp", "--tools=agent"]`
- [ ] `engram.ts` plugin tiene `ENGRAM_BIN` con fallback a `engram` (no Windows path)
- [ ] Git configurado con nombre y email
- [ ] SSH keys configuradas y conectando a GitHub
- [ ] Proyectos clonados en `~/dev/personal/`
- [ ] `omarchy update` ejecutado