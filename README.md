# Waydroid Custom Script ### WCS · Gerenciamento, diagnóstico e personalização do Waydroid [![Versão](https://img.shields.io/badge
vers%C3%A3o-0.5.0-5865F2?style=for-the-badge)](CHANGELOG.md) [![Licença](https://img.shields.io/badge/licen%C3%A7a-GPL--3.0--or-
later-2ea44f?style=for-the-badge)](LICENSE) [![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?style=for-the
badge&logo=python&logoColor=white)](https://www.python.org/) [![Waydroid](https://img.shields.io/badge/plataforma-Waydroid-1f6feb
style=for-the-badge)](https://waydro.id/) **Uma caixa de ferramentas para administrar imagens, componentes e recursos do Waydroid em
Linux.** [Recursos](#-recursos) · [Instalação](#-instalação) · [Comandos](#-guia-rápido) · [Compatibilidade](#-compatibilidade) ·
[Documentação](#-documentação) 
 ✦ Sobre o projeto 
O Waydroid Custom Script (WCS) é uma ferramenta independente, escrita em Python, para inspecionar e gerenciar instalações do Waydroid.
Reúne em uma única interface operações que normalmente exigiriam vários comandos e procedimentos separados. 
O projeto oferece catálogo de imagens, diagnóstico, backups, gerenciamento de aplicativos e módulos, integração experimental com Magisk e
instaladores de componentes. Também reconhece imagens oficiais e imagens customizadas, sem presumir que todas tenham a mesma versão
do Android ou a mesma estrutura. 
[!IMPORTANT] Compatibilidade não significa garantia de execução. O WCS valida pré-condições, arquivos e configurações, mas
recursos que dependem do boot Android, ART, Magisk, spoofing de assinatura, DRM ou execução de APKs precisam ser testados
na imagem real. As limitações conhecidas estão descritas neste README e em docs/COMPATIBILITY.md. 
✨ Recursos### 🖼️ Imagens e OTA - Catálogo de manifestos OTA oficiais. -
 ### 🧰 Sistema e manutenção - Diagnóstico do host e do Waydroid. -
LineageOS e Bliss. - Variantes VANILLA, FOSS e GAPPS. - Detecção de
 Detecção de Android/API e arquitetura. - Backup e restauração de
imagens locais e customizadas. - Verificação de arquivos e hashes. -
 `system.img` e `vendor.img`. - Inspeção de extras Android. -
Troca de imagem com backup prévio. Diagnóstico somente leitura de microG. - Pontos de recuperação.### 📦 Aplicativos e módulos - Instalação, listagem e remoção de APKs.
### ⚙️ Native Bridge ARM → x86_64 - libndk para alvos validados. -
- Gerenciamento de módulos Magisk. - Catálogo de componentes. -
 libhoudini para alvos validados. - Digitalis/Berberis para um alvo QPR2
Instalação e remoção controlada de recursos. específico. - Verificação de payload e compatibilidade. - Backup,
rollback e restauração do estado gerenciado. 
📋 Compatibilidade em resumo 
A detecção de versão Android usa o SDK real (ro.build.version.sdk) quando o guest está acessível. O catálogo OTA consulta os manifestos
publicados; uma combinação não é exibida apenas por existir na tabela abaixo.Área Compatibilidade / condiçãoRuntime do WCS Python 3.10 ou superior; biblioteca padrãoImagens oficiais Canais Waydroid LineageOS/Bliss; VANILLA/FOSS/GAPPS conforme manifesto disponívelAndroid detectável API 26–37 (Android 8–17)libndk / libhoudini Android 11 e 13, host/guest x86_64 e overlay habilitadoDigitalis/Berberis WayDroid-ATV LineageOS 23.2, Android 16 QPR2, release 20260717, host/guest x86_64 e overlay habilitadoMagisk adaptado Experimental; API 30–37 e ABIs documentadas, validação real por imagem necessáriamicroG DresOS API 26–36, imagem sem GApps e Magisk; spoofing/login precisam de validação no Android 
[!WARNING] Não force um Native Bridge em uma imagem fora da matriz. libndk/libhoudini não possuem payload validado nesta
versão para Android 16/17. O bundle Digitalis/Berberis é específico do Android 16 QPR2 indicado acima, não um pacote genérico
para qualquer Android 16 ou Android 17. 
Consulte a matriz detalhada de compatibilidade antes de instalar componentes. 
🚀 Instalação 
Requisitos 
Linux com Waydroid instalado e inicializado.
Python 3.10 ou superior.
bash e utilitários padrão do sistema.
sudo para operações que alteram imagens, overlays ou arquivos do sistema.
Waydroid parado para operações de escrita no overlay ou troca de imagem. 
Instalar no sistema 
Extraia o ZIP, entre no diretório wcs e execute: 
sudo bash install.sh
Por padrão, o instalador coloca os arquivos em /usr/local/lib/wcs e cria o comando /usr/local/bin/wcs. 
Confirme a instalação: 
wcs --version
wcs --help

Executar sem instalar 
Também é possível executar diretamente do diretório extraído: 
./wcs --help
./wcs info
 Use sudo nos comandos que exigem privilégios administrativos. 
🧭 Guia rápido 
Diagnóstico 
wcs info
wcs doctor
wcs compatibility --android 13
wcs extras
wcs microg

Consultar imagens 
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

Trocar imagem oficial 
sudo wcs images switch --rom lineage --type VANILLA
sudo wcs images switch --rom bliss --type FOSS --yes
sudo wcs images switch --rom lineage --type GAPPS --yes
 A operação exige as pré-condições do Waydroid, cria backup da dupla system.img/vendor.img e delega o download e a validação OTA ao
inicializador oficial waydroid init. O WCS não inventa URLs ou nomes de imagens. 
[!CAUTION] O backup da troca de imagem inclui somente system.img e vendor.img. Ele não inclui /data, aplicativos, contas ou
dados pessoais. A troca pode recriar a camada writable do overlay. A integração Magisk gerenciada pelo WCS precisa ser removida
antes da troca. 
Backup e restauração 
sudo wcs backup create
sudo wcs backup list
sudo wcs backup restore NOME_DO_BACKUP

Aplicativos Android 
wcs apps listwcs apps install ~/Downloads/aplicativo.apk
wcs apps remove com.exemplo.aplicativo

Módulos Magisk 
sudo wcs modules list
sudo wcs modules install ./modulo.zip
sudo wcs modules enable modulo_id
sudo wcs modules disable modulo_id
sudo wcs modules remove modulo_id
 Essas operações dependem de uma instalação Magisk funcional dentro do guest. A aceitação de um ZIP pelo host não comprova que o módulo
funcionará após o boot. 
🧩 Catálogo de componentes 
O catálogo reúne os componentes herdados do Waydroid Script e os providers de tradução ARM. O estado e as restrições são verificados pelo
WCS antes da operação.Componente Identificador ObservaçõesGoogle Apps / variante de imagem gapps Seleção de imagem oficial; depende de manifesto disponívelmicroG + Aurora microg Módulo systemless; API 26–36, sem GApps, requer Magisk e spoofing da ROMlibndk ARM translation libndk Android 11/13 x86_64; provider exclusivolibhoudini ARM translation libhoudini Android 11/13 x86_64; provider exclusivoMagisk adaptado magisk Integração experimental; boot/root por imagem não são garantidosWidevine DRM L3 widevine Detecção; não instala blobs genéricosSmart Dock smartdock APK instalado como aplicativo Android normalF-Droid Privileged Extension fdroidpriv Módulo Magisk experimentalCertificado CA personalizado mitm Módulo Magisk experimental para certificado fornecido pelo usuárioHack nodataperm nodataperm Legado; bloqueado quando incompatívelOcultar barra de status hidestatusbar Legado; bloqueado quando incompatívelDigitalis/Berberis digitalis Provider adicional, restrito ao WayDroid-ATV Android 16 QPR2 especificado 
Instalar ou remover um componente: 
sudo wcs features
sudo wcs install microg
sudo wcs remove microg

Sobre os tradutores ARM 
libndk, libhoudini e digitalis são providers distintos. A propriedade Native Bridge do Android seleciona um provider; por isso, o WCS não
mantém esses três mecanismos ativos simultaneamente. 
libndk e libhoudini: utilizam payloads upstream fixados para Android 11/13 x86_64. O instalador substitui a Native Bridge declarada pela
imagem através do overlay, registra o estado anterior e permite restauração.
Digitalis/Berberis: integra um bundle específico do backend Berberis para WayDroid-ATV LineageOS 23.2 / Android 16 QPR2 (release
20260717). O bundle possui 74 artefatos fixados e verificados individualmente por SHA-256. Não é intercambiável com QPR0, Android 16
genérico ou Android 17. 
Exemplos: 
sudo wcs install libndk
sudo wcs install libhoudini
sudo wcs install digitalis
 # Remover o provider e restaurar o estado anterior
sudo wcs remove digitalis
 O Waydroid deve estar parado e a imagem precisa usar overlay. A troca entre providers restaura o estado-base gerenciado antes de aplicar o
novo. A tradução de instruções não fornece aceleração gráfica, Vulkan ou compatibilidade universal com jogos. Boot, inicialização do ART e
abertura de APKs ARM/ARM64 precisam ser testados no guest real. 
🔐 Magisk, microG e autenticaçãoO WCS inclui integração experimental com Magisk e diagnóstico de microG. O diagnóstico wcs microg é somente leitura: coleta evidências de
pacotes, caminhos privilegiados, allowlists, WebView/Custom Tabs e GServices. 
Essas evidências não comprovam autenticação Google. Para validar microG, é necessário conferir o Self-Check do próprio microG,
especialmente a spoofing de assinatura, e testar registro do dispositivo, login, sincronização e Play Store/Aurora no Android em execução. 
O módulo DresOS não inclui Zygisk/Xposed para fornecer spoofing de assinatura. A ROM precisa oferecer o mecanismo compatível. Consulte
docs/COMPATIBILITY.md para restrições por API e imagem. 
🗂️ Onde os dados ficamCaminho Finalidade/usr/local/lib/wcs Instalação padrão do programa/usr/local/bin/wcs Link do comando wcs/var/lib/wcs Estado, backups e cache do WCS/var/lib/wcs/arm-translation Estado/backup transacional dos providers ARM/var/lib/wcs/cache/digitalis-qpr2 Cache do bundle Digitalis QPR2/var/lib/waydroid Dados e configuração do Waydroid, não pertencentes ao WCS 
Os caminhos de estado podem ser afetados pela configuração do sistema. Não apague /var/lib/waydroid como método de desinstalação
do WCS: isso remove dados do próprio Waydroid. 
🧱 Estrutura do projeto 
wcs/
├── bin/├── config/├── core/│ ├── arm_translation.py│ ├── digitalis_translation.py│ ├── image_manager.py│ ├── backup.py│ ├── feature_manager.py│ ├── module_manager.py│ ├── root_manager.py│ └── ...
├── docs/├── modules/├── root/├── tests/├── utils/├── install.sh├── uninstall.sh├── wcs├── CHANGELOG.md├── LICENSE└── THIRD_PARTY_NOTICES.md
# Recursos auxiliares
# Configurações e padrões
# Implementação Python
# libndk e libhoudini
# Digitalis/Berberis
# Imagens e OTA
# Backups e restauração
# Catálogo de componentes
# Módulos Magisk
# Integração Magisk
# Compatibilidade, pesquisa e plano de testes
# Recursos de módulos
# Recursos de root
# Testes automatizados
# Utilitários
# Instalação
# Desinstalação
# Entrada CLI
# Histórico de versões
# Licença
# Avisos de terceiros
🧪 Testes e verificação 
A suíte automatizada pode ser executada a partir da raiz do projeto: 
python3 -m unittest discover -s tests -v
 Os testes cobrem parsing, detecção, pré-condições, integridade de arquivos, operações simuladas de overlay, rollback e regressões conhecidas.
Eles não substituem testes de integração em uma instalação Waydroid real. 
Não são garantidos apenas pelos testes de host: boot Android, funcionamento de MagiskSU/Zygisk/LSPosed, spoofing de assinatura, login
Google, Widevine/DRM, inicialização do ART ou abertura de aplicativos e jogos ARM64. 
📚 DocumentaçãoDocumento Conteúdodocs/COMPATIBILITY.md Matriz de compatibilidade, limites por Android e Native Bridgedocs/TEST-PLAN.md Plano e critérios de testesdocs/RESEARCH.mddocs/LEGACY-PARITY.md Paridade dos componentes legadosCHANGELOG.md Histórico de alteraçõesTHIRD_PARTY_NOTICES.md Créditos e avisos de terceiros
🤝 Contribuição 
Contribuições são bem-vindas. Para relatar um defeito, inclua, quando possível: 
Versão do WCS e distribuição Linux.
Versão/API, ROM, arquitetura e variante da imagem Waydroid.
Comando executado e mensagem de erro completa.
Resultado esperado e resultado observado.
Se o problema ocorre após reiniciar o contêiner. 
Evite publicar dados pessoais, tokens, chaves privadas, certificados privados ou logs que contenham informações sensíveis. 
⚖️ Licença e terceiros 
O WCS é distribuído sob GNU General Public License v3.0 ou posterior. Consulte LICENSE e THIRD_PARTY_NOTICES.md para os termos e
créditos dos componentes de terceiros. 
 **Waydroid Custom Script · WCS 0.5.0** *Ferramentas claras. Alterações verificáveis. Compatibilidade sem promessas falsas.*
