<div align="center">

# Waydroid Custom Script

### WCS · Gerenciamento, diagnóstico e personalização do Waydroid

[![Versão](https://img.shields.io/badge/vers%C3%A3o-0.5.0-5865F2?style=for-the-badge)](CHANGELOG.md)
[![Licença](https://img.shields.io/badge/licen%C3%A7a-GPL--3.0--or--later-2ea44f?style=for-the-badge)](LICENSE)
[![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![Waydroid](https://img.shields.io/badge/plataforma-Waydroid-1f6feb?style=for-the-badge)](https://waydro.id/)

**Uma caixa de ferramentas para administrar imagens, componentes e recursos do Waydroid em Linux.**

[Recursos](#-recursos) · [Instalação](#-instalação) · [Comandos](#-guia-rápido) · [Compatibilidade](#-compatibilidade) · [Documentação](#-documentação)

</div>

---

## ✦ Sobre o projeto

O **Waydroid Custom Script (WCS)** é uma ferramenta independente, escrita em Python, para inspecionar e gerenciar instalações do [Waydroid](https://waydro.id/). Reúne em uma única interface operações que normalmente exigiriam vários comandos e procedimentos separados.

O projeto oferece catálogo de imagens, diagnóstico, backups, gerenciamento de aplicativos e módulos, integração experimental com Magisk e instaladores de componentes. Também reconhece imagens oficiais e imagens customizadas, sem presumir que todas tenham a mesma versão do Android ou a mesma estrutura.

> [!IMPORTANT]
> **Compatibilidade não significa garantia de execução.** O WCS valida pré-condições, arquivos e configurações, mas recursos que dependem do boot Android, ART, Magisk, spoofing de assinatura, DRM ou execução de APKs precisam ser testados na imagem real. As limitações conhecidas estão descritas neste README e em [`docs/COMPATIBILITY.md`](docs/COMPATIBILITY.md).

## ✨ Recursos

<table>
<tr>
<td width="50%">

### 🖼️ Imagens e OTA
- Catálogo de manifestos OTA oficiais.
- LineageOS e Bliss.
- Variantes VANILLA, FOSS e GAPPS.
- Detecção de imagens locais e customizadas.
- Verificação de arquivos e hashes.
- Troca de imagem com backup prévio.

</td>
<td width="50%">

### 🧰 Sistema e manutenção
- Diagnóstico do host e do Waydroid.
- Detecção de Android/API e arquitetura.
- Backup e restauração de `system.img` e `vendor.img`.
- Inspeção de extras Android.
- Diagnóstico somente leitura de microG.
- Pontos de recuperação.

</td>
</tr>
<tr>
<td width="50%">

### 📦 Aplicativos e módulos
- Instalação, listagem e remoção de APKs.
- Gerenciamento de módulos Magisk.
- Catálogo de componentes.
- Instalação e remoção controlada de recursos.

</td>
<td width="50%">

### ⚙️ Native Bridge ARM → x86_64
- libndk para alvos validados.
- libhoudini para alvos validados.
- Digitalis/Berberis para um alvo QPR2 específico.
- Verificação de payload e compatibilidade.
- Backup, rollback e restauração do estado gerenciado.

</td>
</tr>
</table>

## 📋 Compatibilidade em resumo

A detecção de versão Android usa o SDK real (`ro.build.version.sdk`) quando o guest está acessível. O catálogo OTA consulta os manifestos publicados; uma combinação não é exibida apenas por existir na tabela abaixo.

| Área | Compatibilidade / condição |
|---|---|
| Runtime do WCS | Python 3.10 ou superior; biblioteca padrão |
| Imagens oficiais | Canais Waydroid LineageOS/Bliss; VANILLA/FOSS/GAPPS conforme manifesto disponível |
| Android detectável | API 26–37 (Android 8–17) |
| libndk / libhoudini | Android 11 e 13, host/guest x86_64 e overlay habilitado |
| Digitalis/Berberis | WayDroid-ATV LineageOS 23.2, Android 16 QPR2, release 20260717, host/guest x86_64 e overlay habilitado |
| Magisk adaptado | Experimental; API 30–37 e ABIs documentadas, validação real por imagem necessária |
| microG DresOS | API 26–36, imagem sem GApps e Magisk; spoofing/login precisam de validação no Android |

> [!WARNING]
> **Não force um Native Bridge em uma imagem fora da matriz.** libndk/libhoudini não possuem payload validado nesta versão para Android 16/17. O bundle Digitalis/Berberis é específico do Android 16 QPR2 indicado acima, não um pacote genérico para qualquer Android 16 ou Android 17.

Consulte a [matriz detalhada de compatibilidade](docs/COMPATIBILITY.md) antes de instalar componentes.

## 🚀 Instalação

### Requisitos

- Linux com Waydroid instalado e inicializado.
- Python 3.10 ou superior.
- `bash` e utilitários padrão do sistema.
- `sudo` para operações que alteram imagens, overlays ou arquivos do sistema.
- Waydroid parado para operações de escrita no overlay ou troca de imagem.

### Instalar no sistema

Extraia o ZIP, entre no diretório `wcs` e execute:

```bash
sudo bash install.sh
```

Por padrão, o instalador coloca os arquivos em `/usr/local/lib/wcs` e cria o comando `/usr/local/bin/wcs`.

Confirme a instalação:

```bash
wcs --version
wcs --help
```

### Executar sem instalar

Também é possível executar diretamente do diretório extraído:

```bash
./wcs --help
./wcs info
```

Use `sudo` nos comandos que exigem privilégios administrativos.

## 🧭 Guia rápido

### Diagnóstico

```bash
wcs info
wcs doctor
wcs compatibility --android 13
wcs extras
wcs microg
```

### Consultar imagens

```bash
# Catálogo disponível para a arquitetura detectada
wcs images catalog

# Consultar todos os alvos publicados
wcs images catalog --arch all --rom all --type all

# Ver canais OTA da instalação atual
wcs images ota

# Verificar imagens locais
wcs images verify

# Catálogo separado de releases WayDroid-ATV
wcs images atv-catalog --android all
```

### Trocar imagem oficial

```bash
sudo wcs images switch --rom lineage --type VANILLA
sudo wcs images switch --rom bliss --type FOSS --yes
sudo wcs images switch --rom lineage --type GAPPS --yes
```

A operação exige as pré-condições do Waydroid, cria backup da dupla `system.img`/`vendor.img` e delega o download e a validação OTA ao inicializador oficial `waydroid init`. O WCS não inventa URLs ou nomes de imagens.

> [!CAUTION]
> O backup da troca de imagem inclui **somente `system.img` e `vendor.img`**. Ele não inclui `/data`, aplicativos, contas ou dados pessoais. A troca pode recriar a camada writable do overlay. A integração Magisk gerenciada pelo WCS precisa ser removida antes da troca.

### Backup e restauração

```bash
sudo wcs backup create
sudo wcs backup list
sudo wcs backup restore NOME_DO_BACKUP
```

### Aplicativos Android

```bash
wcs apps list
wcs apps install ~/Downloads/aplicativo.apk
wcs apps remove com.exemplo.aplicativo
```

### Módulos Magisk

```bash
sudo wcs modules list
sudo wcs modules install ./modulo.zip
sudo wcs modules enable modulo_id
sudo wcs modules disable modulo_id
sudo wcs modules remove modulo_id
```

Essas operações dependem de uma instalação Magisk funcional dentro do guest. A aceitação de um ZIP pelo host não comprova que o módulo funcionará após o boot.

## 🧩 Catálogo de componentes

O catálogo reúne os componentes herdados do Waydroid Script e os providers de tradução ARM. O estado e as restrições são verificados pelo WCS antes da operação.

| Componente | Identificador | Observações |
|---|---|---|
| Google Apps / variante de imagem | `gapps` | Seleção de imagem oficial; depende de manifesto disponível |
| microG + Aurora | `microg` | Módulo systemless; API 26–36, sem GApps, requer Magisk e spoofing da ROM |
| libndk ARM translation | `libndk` | Android 11/13 x86_64; provider exclusivo |
| libhoudini ARM translation | `libhoudini` | Android 11/13 x86_64; provider exclusivo |
| Magisk adaptado | `magisk` | Integração experimental; boot/root por imagem não são garantidos |
| Widevine DRM L3 | `widevine` | Detecção; não instala blobs genéricos |
| Smart Dock | `smartdock` | APK instalado como aplicativo Android normal |
| F-Droid Privileged Extension | `fdroidpriv` | Módulo Magisk experimental |
| Certificado CA personalizado | `mitm` | Módulo Magisk experimental para certificado fornecido pelo usuário |
| Hack nodataperm | `nodataperm` | Legado; bloqueado quando incompatível |
| Ocultar barra de status | `hidestatusbar` | Legado; bloqueado quando incompatível |
| Digitalis/Berberis | `digitalis` | Provider adicional, restrito ao WayDroid-ATV Android 16 QPR2 especificado |

Instalar ou remover um componente:

```bash
sudo wcs features
sudo wcs install microg
sudo wcs remove microg
```

### Sobre os tradutores ARM

`libndk`, `libhoudini` e `digitalis` são **providers distintos**. A propriedade Native Bridge do Android seleciona um provider; por isso, o WCS não mantém esses três mecanismos ativos simultaneamente.

- **libndk e libhoudini:** utilizam payloads upstream fixados para Android 11/13 x86_64. O instalador substitui a Native Bridge declarada pela imagem através do overlay, registra o estado anterior e permite restauração.
- **Digitalis/Berberis:** integra um bundle específico do backend Berberis para WayDroid-ATV LineageOS 23.2 / Android 16 QPR2 (release 20260717). O bundle possui 74 artefatos fixados e verificados individualmente por SHA-256. Não é intercambiável com QPR0, Android 16 genérico ou Android 17.

Exemplos:

```bash
sudo wcs install libndk
sudo wcs install libhoudini
sudo wcs install digitalis

# Remover o provider e restaurar o estado anterior
sudo wcs remove digitalis
```

O Waydroid deve estar parado e a imagem precisa usar overlay. A troca entre providers restaura o estado-base gerenciado antes de aplicar o novo. A tradução de instruções não fornece aceleração gráfica, Vulkan ou compatibilidade universal com jogos. Boot, inicialização do ART e abertura de APKs ARM/ARM64 precisam ser testados no guest real.

## 🔐 Magisk, microG e autenticação

O WCS inclui integração experimental com Magisk e diagnóstico de microG. O diagnóstico `wcs microg` é somente leitura: coleta evidências de pacotes, caminhos privilegiados, allowlists, WebView/Custom Tabs e GServices.

Essas evidências **não comprovam autenticação Google**. Para validar microG, é necessário conferir o Self-Check do próprio microG, especialmente a spoofing de assinatura, e testar registro do dispositivo, login, sincronização e Play Store/Aurora no Android em execução.

O módulo DresOS não inclui Zygisk/Xposed para fornecer spoofing de assinatura. A ROM precisa oferecer o mecanismo compatível. Consulte [`docs/COMPATIBILITY.md`](docs/COMPATIBILITY.md) para restrições por API e imagem.

## 🗂️ Onde os dados ficam

| Caminho | Finalidade |
|---|---|
| `/usr/local/lib/wcs` | Instalação padrão do programa |
| `/usr/local/bin/wcs` | Link do comando `wcs` |
| `/var/lib/wcs` | Estado, backups e cache do WCS |
| `/var/lib/wcs/arm-translation` | Estado/backup transacional dos providers ARM |
| `/var/lib/wcs/cache/digitalis-qpr2` | Cache do bundle Digitalis QPR2 |
| `/var/lib/waydroid` | Dados e configuração do Waydroid, não pertencentes ao WCS |

Os caminhos de estado podem ser afetados pela configuração do sistema. Não apague `/var/lib/waydroid` como método de desinstalação do WCS: isso remove dados do próprio Waydroid.

## 🧱 Estrutura do projeto

```text
wcs/
├── bin/                         # Recursos auxiliares
├── config/                      # Configurações e padrões
├── core/                        # Implementação Python
│   ├── arm_translation.py       # libndk e libhoudini
│   ├── digitalis_translation.py # Digitalis/Berberis
│   ├── image_manager.py         # Imagens e OTA
│   ├── backup.py                # Backups e restauração
│   ├── feature_manager.py       # Catálogo de componentes
│   ├── module_manager.py        # Módulos Magisk
│   ├── root_manager.py          # Integração Magisk
│   └── ...
├── docs/                        # Compatibilidade, pesquisa e plano de testes
├── modules/                     # Recursos de módulos
├── root/                        # Recursos de root
├── tests/                       # Testes automatizados
├── utils/                       # Utilitários
├── install.sh                   # Instalação
├── uninstall.sh                 # Desinstalação
├── wcs                          # Entrada CLI
├── CHANGELOG.md                 # Histórico de versões
├── LICENSE                      # Licença
└── THIRD_PARTY_NOTICES.md       # Avisos de terceiros
```

## 🧪 Testes e verificação

A suíte automatizada pode ser executada a partir da raiz do projeto:

```bash
python3 -m unittest discover -s tests -v
```

Os testes cobrem parsing, detecção, pré-condições, integridade de arquivos, operações simuladas de overlay, rollback e regressões conhecidas. Eles não substituem testes de integração em uma instalação Waydroid real.

**Não são garantidos apenas pelos testes de host:** boot Android, funcionamento de MagiskSU/Zygisk/LSPosed, spoofing de assinatura, login Google, Widevine/DRM, inicialização do ART ou abertura de aplicativos e jogos ARM64.

## 📚 Documentação

| Documento | Conteúdo |
|---|---|
| [`docs/COMPATIBILITY.md`](docs/COMPATIBILITY.md) | Matriz de compatibilidade, limites por Android e Native Bridge |
| [`docs/TEST-PLAN.md`](docs/TEST-PLAN.md) | Plano e critérios de testes |
| [`docs/RESEARCH.md`](docs/RESEARCH.md) | Referências e pesquisa técnica |
| [`docs/LEGACY-PARITY.md`](docs/LEGACY-PARITY.md) | Paridade dos componentes legados |
| [`CHANGELOG.md`](CHANGELOG.md) | Histórico de alterações |
| [`THIRD_PARTY_NOTICES.md`](THIRD_PARTY_NOTICES.md) | Créditos e avisos de terceiros |

## 🤝 Contribuição

Contribuições são bem-vindas. Para relatar um defeito, inclua, quando possível:

1. Versão do WCS e distribuição Linux.
2. Versão/API, ROM, arquitetura e variante da imagem Waydroid.
3. Comando executado e mensagem de erro completa.
4. Resultado esperado e resultado observado.
5. Se o problema ocorre após reiniciar o contêiner.

Evite publicar dados pessoais, tokens, chaves privadas, certificados privados ou logs que contenham informações sensíveis.

## ⚖️ Licença e terceiros

O WCS é distribuído sob **GNU General Public License v3.0 ou posterior**. Consulte [`LICENSE`](LICENSE) e [`THIRD_PARTY_NOTICES.md`](THIRD_PARTY_NOTICES.md) para os termos e créditos dos componentes de terceiros.

---

<div align="center">

**Waydroid Custom Script · WCS 0.5.0**  
*Ferramentas claras. Alterações verificáveis. Compatibilidade sem promessas falsas.*

</div>
