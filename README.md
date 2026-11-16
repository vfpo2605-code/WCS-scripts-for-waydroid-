Waydroid Custom Script (WCS) 
Versão: 0.5.0
Licença do WCS: GPL-3.0-or-later
Runtime: Python 3.10 ou superior, usando a biblioteca padrão
Plataforma: Linux com Waydroid instalado 
O Waydroid Custom Script (WCS) é uma ferramenta de administração e manutenção de instalações Waydroid. Ele reúne inspeção do ambiente,
catálogo de imagens, backup e restauração de imagens, integração experimental do Magisk, gerenciamento de módulos, instalação de APKs,
diagnóstico de componentes Android e provedores de Native Bridge para executar aplicações ARM em imagens x86/x86_64. 
O WCS procura trabalhar com os mecanismos reais do Waydroid e dos componentes upstream. Ele identifica a imagem instalada antes de
operações sensíveis, verifica arquivos baixados, cria backups quando substitui imagens ou arquivos do overlay e bloqueia operações para as
quais não existe um payload ou uma combinação de compatibilidade documentada. 
Importante: a presença de um recurso no catálogo ou a conclusão de uma instalação de arquivos não prova que o Android
inicializou corretamente, que o MagiskSU/Zygisk está operacional, que o login Google funciona ou que todo aplicativo ARM será
executado. Essas verificações exigem testes dentro da imagem Waydroid de destino. 
 Índice 
Visão geral
Recursos
Requisitos
Instalação
Atualização e desinstalação
Uso rápido
Referência da CLI
Imagens oficiais Waydroid
Backups e restauração
Catálogo de componentes
Native Bridge: libndk, libhoudini e Digitalis
Magisk e módulos
APKs e aplicativos
Diagnóstico, microG e certificação
Arquivos e diretórios utilizados
Segurança e comportamento das operações
Testes e limites conhecidos
Estrutura do projeto
Licenças e créditos 
 Visão geral 
O WCS é um gerenciador do lado Linux. Ele consulta a configuração e o estado do Waydroid, chama o executável waydroid para operações
próprias do contêiner, executa comandos no guest quando necessário e mantém seus backups e registros em um diretório de estado separado.
As principais áreas do projeto são: 
Detecção: consulta versão Android/API, arquitetura, estado da sessão/contêiner/LXC, origem da imagem, canais OTA, caminhos de
imagens, overlay e diretório de dados.
Imagens: consulta manifestos OTA oficiais, lista releases WayDroid-ATV, verifica arquivos locais e troca imagens oficiais através do
inicializador do Waydroid.
Backups: cria cópias verificadas de system.img e vendor.img, com manifesto SHA-256, e restaura um par previamente salvo.
Root: integra experimentalmente Magisk 31.0 ao overlay do Waydroid e oferece diagnóstico e recuperação do estado dessa integração.
Módulos: valida ZIPs Magisk, delega a instalação ao próprio Magisk dentro do Android e gerencia os marcadores de habilitação,
desabilitação e remoção.
Componentes: instala ou remove componentes compatíveis do catálogo herdado do Waydroid Script, sem aplicar automaticamente
patches antigos em imagens não verificadas.
Native Bridge: gerencia libndk, libhoudini e Digitalis/Berberis como provedores alternativos de tradução ARM para hosts x86/x86_64,
cada um com restrições próprias.
Aplicativos e diagnóstico: instala/lista/remove APKs, coleta evidências sobre GApps, microG, Widevine e Native Bridge e obtém o Android
ID quando o GSF está disponível. 
O WCS não instala dependências do sistema operacional por gerenciador de pacotes, não troca o kernel Linux e não configura firewall ou rede
do host.RecursosÁrea Recursos disponíveisDetecção Waydroid, sessão, contêiner, LXC, Android/API, ABI, ROM, tipo de sistema, canais OTA, imagens, overlay e dadosImagens Catálogo oficial, catálogo WayDroid-ATV, consulta de manifestos, verificação local, troca de imagemBackup Criar, listar e restaurar backups de system.img e vendor.imgMagisk Instalar adaptador WCS 31.0, consultar estado, remover integração e recuperar instalação interrompidaMódulos Listar, instalar ZIP, habilitar, desabilitar e agendar remoçãoComponentes GApps/tipo de imagem, microG, libndk, libhoudini, Digitalis, Magisk, Widevine, Smart Dock, F-Droid Privileged, CA MITM e
recursos legados bloqueadosAplicativos Listar, instalar e remover APKsDiagnóstico doctor, compatibilidade, extras Android, microG/autenticação e pontos de recuperaçãoCertificação Consultar Android ID obtido do banco GServices, quando disponível Requisitos 

No Linux host 
Linux com uma instalação Waydroid funcional.
Python 3.10 ou superior.
Bash para os scripts de instalação e desinstalação.
Executável waydroid disponível no PATH.
Acesso à rede para consultar manifestos e baixar componentes upstream.
Permissão sudo para operações que alteram imagens, overlays, dados ou arquivos protegidos do Waydroid. 
O código Python do WCS utiliza apenas módulos da biblioteca padrão. Alguns recursos dependem de ferramentas externas já presentes no
host ou no guest:Ferramenta Usowaydroid Consultar estado, executar comandos no Android, instalar APKs e trocar imagensadb Tentativa adicional de comprovar o estado de runtime do Magisk; não é necessário para todas as funçõesopenssl Validar e preparar o certificado CA do recurso mitmsqlite3 no Android guest Ler o Android ID do banco GServices 
No Android guest 
As exigências variam por recurso. Por exemplo, instalação de módulos precisa de Waydroid em execução e Magisk funcional; instalação de
tradutores exige Waydroid parado e overlay habilitado. 
Não existe uma única combinação de Android/API/ABI que habilite todos os componentes. Consulte Catálogo de componentes e
Compatibilidade. 
 Instalação 
O ZIP contém a pasta wcs/. Extraia-o e entre nessa pasta: 
unzip wcs-0.5.0-digitalis-native-bridge.zip
cd wcs
 Instale no prefixo padrão /usr/local: 
sudo bash install.sh
 O instalador copia o projeto para: 
/usr/local/lib/wcs/
 e cria o comando: 
/usr/local/bin/wcs -> /usr/local/lib/wcs/wcs
 Verifique a instalação: 
wcs --versionwcswcs
--help
doctor
Executar sem instalar 
Também é possível usar a cópia extraída diretamente: 
./wcs --version
./wcs --help
./wcs info
 Para operações administrativas, use sudo: 
sudo ./wcs root status

Instalação em prefixo personalizado 
O install.sh e o uninstall.sh aceitam a variável PREFIX. O padrão é /usr/local. O destino deve ser absoluto e não pode conter
componentes ... 
Exemplo: 
sudo env PREFIX=/opt/wcs bash install.sh
 Nesse caso, o executável será instalado em /opt/wcs/bin/wcs, e os arquivos do programa em /opt/wcs/lib/wcs. 
 Atualização e desinstalação 
Atualizar o código instalado 
Extraia a nova versão e execute novamente: 
cd wcs
sudo bash install.sh
 O instalador substitui as cópias de código, configuração, documentação e testes em PREFIX/lib/wcs. O estado do WCS, os backups e os
dados Android ficam fora dessa pasta e não são removidos pelo instalador. 
Desinstalar somente o programa WCS 
Na pasta extraída da versão instalada: 
sudo bash uninstall.sh
 O desinstalador remove o link PREFIX/bin/wcs e a pasta PREFIX/lib/wcs somente quando o link aponta para o destino esperado. Ele não
apaga: 
/var/lib/wcs (estado, cache e backups);
dados Android;
imagens Waydroid;
módulos Magisk;
modificações já aplicadas ao overlay. 
Desinstalar o executável do WCS não equivale a reverter as modificações aplicadas no Waydroid. Use os comandos próprios de remoção antes
de desinstalar o programa, quando a restauração for necessária. 
 Uso rápido 
# Versão e ajuda
wcs --version
wcs --help
 # Inspeção da instalação e diagnósticowcs info
wcs doctor
 # Catálogo e estado das imagens
wcs images catalog
wcs images ota
wcs images verify
 # Componentes e diagnóstico de extras
wcs features
wcs extras
wcs microg
 # Compatibilidade e pontos de recuperação
wcs compatibility --android 16
wcs recovery
 Executar wcs sem argumentos abre o menu interativo. As opções são:Opção Ação1 Informações do Waydroid2 Catálogo oficial de imagens Waydroid3 Consultar manifestos OTA configurados4 Verificar imagens locais5 Criar backup de system.img/vendor.img6 Listar backups7 Restaurar backup8 Status Magisk/root9 Instalar adaptador Magisk 31.010 Remover integração Magisk WCS11 Gerenciar módulos Magisk12 Diagnóstico do host e Waydroid13 Relatórios de compatibilidade Zygisk/LSPosed14 Diagnóstico de GApps, microG, Widevine e Native Bridge15 Atalhos para gerenciar APKs16 Consultar Android ID do Google Play17 Catálogo e instaladores de componentes18 Remover componente19 Releases WayDroid-ATV20 Trocar imagem oficial21 Diagnóstico somente leitura de microG/autenticação0 Sair 
 Referência da CLI 
Informações e diagnósticoComando Funçãowcs --version Exibe a versãowcs --help Exibe os subcomandos e opçõeswcs info Mostra estado, Android/API, ABI, ROM, canais OTA, caminhos de imagens, overlay e dadoswcs doctor Verifica binário/configuração Waydroid, Python, privilégios, estado, imagens, overlay e diretório de
dadoswcs compatibility --android 13 Exibe a matriz de compatibilidade e as evidências declaradas para a versão escolhidawcs recovery Lista pontos de recuperação disponíveisO argumento --android de compatibility aceita 11, 12, 12L, 13, 14, 15, 16 e 17. O comando wcs uninstall é um alias de wcs remove. 
Imagens 
# Catálogo OTA oficial para a arquitetura do host
wcs images catalog
 # Todas as arquiteturas/ROMs/tipos com manifesto publicado
wcs images catalog --arch all --rom all --type all
 # Filtrar uma combinação
wcs images catalog --arch x86_64 --rom bliss --type GAPPS
 # Catálogo separado de releases WayDroid-ATV
wcs images atv-catalog --android all
 # Inspecionar canais OTA configurados na instalação
wcs images ota
 # Verificar existência e calcular SHA-256 local de system/vendor
wcs images verify
 # Trocar imagem oficial (exige backup e confirmação, salvo --yes)
sudo wcs images switch --rom lineage --type VANILLA
sudo wcs images switch --rom bliss --type FOSS --yes
 Valores aceitos para os filtros: 
ROM: lineage, bliss ou all.
Tipo: VANILLA, FOSS, GAPPS ou all.
Arquitetura: host, all, x86_64, x86, arm64, arm, arm64_only.
O filtro --android 16|17|all em images catalog é usado com --source atv. 
O catálogo oficial consulta manifestos OTA; combinações não publicadas são omitidas. O catálogo ATV lista releases/assets e não é confundido
com a origem OTA oficial do Waydroid. 
images verify calcula fingerprints SHA-256 dos arquivos locais. Quando não há uma comparação com manifesto/checksum oficial, o
resultado não certifica a autenticidade da imagem. 
Backups 
sudo wcs backup create
wcs backup list
sudo wcs backup restore NOME_DO_BACKUP
 Ver Backups e restauração. 
Root / Magisk 
sudo wcs root status
sudo wcs root install --stop
sudo wcs root remove
sudo wcs root recover
 - root install exige Waydroid parado. --stop autoriza o WCS a solicitar a parada quando a instância estiver ativa. - root remove restaura
arquivos originais do overlay pertencentes à integração WCS. - root recover tenta restaurar um backup-base deixado por uma instalação
interrompida. 
A instalação dos arquivos Magisk não comprova boot, daemon, MagiskSU nem Zygisk. Consulte Magisk e módulos. 
Módulos Magisk 
sudo wcs modules list
sudo wcs modules install ./modulo.zip
sudo wcs modules enable modulo_id
sudo wcs modules disable modulo_id
sudo wcs modules remove modulo_id
 Habilitar/desabilitar usa os marcadores reconhecidos pelo Magisk e passa a valer no próximo boot. Remover cria o marcador remove; a
remoção efetiva é processada pelo Magisk durante o próximo boot. 
Aplicativos Android 
wcs apps list
wcs apps install ~/Downloads/aplicativo.apkwcs apps remove com.exemplo.aplicativo
 O WCS valida o APK e delega a instalação ao comando de aplicativo do Waydroid. 
Componentes 
wcs features
sudo wcs install smartdock
sudo wcs install microg
sudo wcs install fdroidpriv
sudo wcs install mitm --ca-cert ~/Downloads/proxy-ca.pem
sudo wcs install gapps --variant GAPPS --rom lineage --yes
sudo wcs install magisk --stop
sudo wcs install libndk
sudo wcs install libhoudini
sudo wcs install digitalis
sudo wcs remove smartdock
sudo wcs remove microg
 O catálogo de componentes também pode ser aberto pelo menu interativo. Nem todo recurso listado é instalável em todas as imagens; alguns
são somente detecção ou permanecem bloqueados até existir um port validado. 
Certificação e extras 
wcs extras
wcs microg
wcs certified
 wcs certified tenta obter o Android ID do Google Services Framework (GSF), quando o pacote/banco está disponível, e informa o endereço
de registro de dispositivos Android não certificados. 
 Imagens oficiais Waydroid 
Canais e variantes reconhecidos 
O WCS reconhece a estrutura dos canais OTA oficiais do Waydroid: 
ROMs: lineage e bliss.
Tipos de sistema: VANILLA, FOSS e GAPPS.
Arquiteturas catalogadas: x86_64, x86, arm64, arm e arm64_only, quando o manifesto correspondente existe.
APIs Android mapeadas na detecção: API 26–37 (Android 8.0–17). 
Os canais-base utilizados pelo inicializador são: 
Sistema: https://ota.waydro.id/system
Vendor: https://ota.waydro.id/vendor 
O WCS consulta manifestos e delega a seleção/instalação da imagem ao inicializador do próprio Waydroid. O Waydroid resolve a arquitetura
apropriada para o host, seleciona o vendor correspondente e valida os arquivos de OTA conforme os hashes publicados nos manifestos. 
Troca de imagem 
wcs images switch exige: 
Waydroid parado;
system.img e vendor.img existentes para o backup;
versão do waydroid init que exponha as opções necessárias (--images_path, --system_channel, --vendor_channel, -
rom_type, --system_type);
diretório de imagens gerenciável por OTA, não uma pasta especial de imagens pré-instaladas;
integração Magisk WCS removida do overlay antes da troca. 
Antes de chamar waydroid init -f, o WCS cria backup verificado do par system.img/vendor.img. O init -f oficial baixa, valida e extrai o
par. O WCS não executa um waydroid upgrade extra depois da troca oficial. 
Atenção: o backup não inclui /data, contas, aplicativos, configurações pessoais ou os dados internos do Android. O inicializador oficial pode
recriar a camada writable do overlay.Imagens pré-instaladas e customizadas 
Imagens localizadas nos caminhos especiais /etc/waydroid-extra/images ou /usr/share/waydroid-extra/images são identificadas
como pré-instaladas. Nessas instalações, a troca automática por OTA é bloqueada porque o Waydroid pode priorizar o par de imagens local e
ignorar os canais OTA. 
Imagens customizadas ainda podem ser inspecionadas, ter seus hashes locais calculados e receber backup quando os arquivos e diretórios
necessários estiverem presentes. O WCS não inventa a origem ou a versão de uma imagem cujo canal não informa esses dados. 
 Backups e restauração 
Os backups de imagem são armazenados em: 
/var/lib/wcs/backups/images/
 Cada backup contém: 
system.img;
vendor.img;
manifest.json com tamanho e SHA-256 dos dois arquivos, data UTC e caminho de origem. 
A criação e a restauração exigem Waydroid parado. A restauração valida o manifesto, tamanho e SHA-256 dos dois arquivos antes de substituir
as imagens. Antes da substituição, as imagens atualmente instaladas também são salvas em um novo backup. 
Escopo do backup: somente system.img e vendor.img. Não inclui /data, diretório de aplicativos, banco GServices, contas, downloads ou
configurações pessoais do Android. 
A integração Magisk e os Native Bridges possuem snapshots próprios dos caminhos que o WCS altera no overlay, guardados no estado do WCS.
Esses snapshots são separados dos backups de imagem. 
 Catálogo de componentes 
O catálogo atualbloqueado.IDgappsmicroglibndklibhoudinidigitalismagiskwidevinesmartdockfdroidprivcontém 12 IDs. O estado indica se o componente pode ser instalado, se é experimental, se é apenas detectado ou se está
Nome Estado nesta versãoGoogle Apps / tipo de imagem OFFICIAL_OTA_SWITCHmicroG + componentes Aurora MAGISK_API26_TO36_VANILLAGoogle/Guybrush ARM translation INSTALLABLE_ANDROID11_13_X86_64Intel/WSA ARM translation INSTALLABLE_ANDROID11_13_X86_64Digitalis/Berberis ARM64 translation EXPERIMENTAL_ANDROID16_QPR2_WAYDR
ID_ATVMagisk adaptado para Waydroid EXPERIMENTAL_API30_TO37Widevine DRM L3 DETECT_ONLYSmart Dock INSTALLABLE_API26_TO37Observações e limitesTroca a imagem oficial entre VANILLA, FOSS
e GAPPS, em lineage ou bliss; depende
de manifesto publicado e pré-condições da
troca.Módulo systemless DresOS. Requer Magisk,
imagem sem GApps, API 26–36 e ROM com
suporte à assinatura falsa. API 37 é
bloqueada.Payload upstream fixado para Android
11/13, host e guest x86_64, overlay
habilitado. Alternativo ao Houdini.Payload upstream fixado para Android
11/13, host e guest x86_64, overlay
habilitado. O instalador gera houdini.rc.
Alternativo ao libndk.Bundle específico WayDroid-ATV LineageOS
23.2 / Android 16 QPR2, build identificada
como 20260717; host/guest x86_64. Não é
tradutor universal para Android 16/17.Adaptador WCS da versão Magisk 31.0 com
APK e patches fixados. A presença dos
arquivos não comprova boot nem runtime.Detecta bibliotecas existentes; não instala
blobs universais nem prova reprodução
DRM licenciada.Obtém APK assinado na origem F-Droid e
instala como aplicativo normal. Não
concede permissões privilegiadas.mitm Certificado CA personalizadonodataperm Hack nodatapermhidestatusbar Ocultar barra de status7INSTALLABLE_EXPERIMENTAL_API30_TO
7LEGACY_NOT_PORTEDLEGACY_NOT_PORTEDsystemless. Requer Magisk e reinicialização
para validar o efeito.Prepara um módulo Magisk para a trust
store Conscrypt APEX. Requer certificado
PEM, openssl, Magisk e validação após
reboot.Bloqueado. O patch antigo substitui
services.jar e depende de builds
específicos do framework.Bloqueado. O overlay legado não tem
matriz de builds/assinaturas validada para
as imagens atuais. 
Comportamento de remoção 
smartdock: solicita a remoção do pacote Android cu.axel.smartdock.
microg: agenda a remoção do módulo Magisk dresosmicrog; o Magisk processa o marcador no próximo boot.
fdroidpriv: agenda a remoção do módulo wcsfdroidpriv.
mitm: agenda a remoção do módulo wcsmitmca.
libndk, libhoudini e digitalis: restauram os arquivos/propriedades anteriores salvos pelo WCS, com Waydroid parado.
magisk: use wcs root remove para restaurar os arquivos do overlay. Isso não apaga /data/adb nem os módulos Magisk instalados.
gapps: não é removido como APK/módulo. Para sair de GAPPS, troque para VANILLA ou FOSS usando o fluxo de troca de imagem e seu
backup.
widevine: é gerenciado pela imagem; o WCS não remove bibliotecas base.
nodataperm e hidestatusbar: não foram instalados pelo WCS nesta versão. 
 Native Bridge: libndk, libhoudini e Digitalis 
Os três componentes são provedores alternativos, não camadas para ativar simultaneamente. O Android escolhe um Native Bridge através de
ro.dalvik.vm.native.bridge. Instalar o segundo provider gerenciado pelo WCS restaura primeiro o estado-base registrado e aplica o novo
provider. 
libndk 
Origem: payload Google/Guybrush mantido em repositório upstream referenciado pelo instalador.
Builds fixados nesta versão: Android 11 e Android 13.
Requer host x86_64, guest x86_64, overlay habilitado e Waydroid parado.
O download é validado pelo MD5 fixado no código do instalador.
O WCS registra arquivos e valores anteriores para permitir remoção/restauração. 
libhoudini 
Origem: payload Intel/Houdini derivado do Windows Subsystem for Android, via repositório upstream referenciado pelo instalador.
Builds fixados nesta versão: Android 11 e Android 13.
Requer host x86_64, guest x86_64, overlay habilitado e Waydroid parado.
O WCS valida o arquivo compactado, a estrutura e os arquivos obrigatórios e gera o hook system/etc/init/houdini.rc com os
registros binfmt_misc usados pelo instalador.
O download é validado pelo MD5 fixado no código do instalador. 
Digitalis / Berberis 
Digitalis é um provider diferente, baseado no backend aberto Berberis do AOSP. Ele não é uma fusão dos binários proprietários libndk/Houdini. 
O WCS integra um bundle precompilado com 74 artefatos para WayDroid-ATV LineageOS 23.2 / Android 16 QPR2, identificado pelo canal a16
qpr2 ou pela imagem pré-instalada release 20260717. 
O bundle contém, entre outros componentes: 
Native Bridge x86-64 libberberis_arm64.so;
runners e registros binfmt_misc;
linker e app_process64 ARM64;
bibliotecas Bionic ARM64 do guest;
proxies de bibliotecas Android;
ld.config.arm64.txt e hook berberis.rc. 
O WCS baixa os arquivos na instalação, valida o manifesto Git fixado e o SHA-256 de cada artefato e confere a arquitetura ELF dos executáveisF-Droid Privileged Extension INSTALLABLE_EXPERIMENTAL_API26_TO
 Converte o OTA oficial em módulo Magisk
bibliotecas principais antes de modificar o overlay. 
Restrições rígidas do Digitalis nesta versão: 
Android 16 QPR2 específico; não é Android 16 genérico/QPR0.
Não aplicar em Android 17.
ROM LineageOS/WayDroid-ATV do bundle, host e guest x86_64.
mount_overlays=True.
Waydroid parado durante a operação. 
Na instalação, o WCS substitui o provider ativo no overlay, mascara o init hook concorrente de ndk_translation, atualiza o seletor Native
Bridge em [waydroid], [properties] e /data/misc/waydroid_settings e executa waydroid upgrade --offline para regenerar a
configuração derivada. A remoção tenta restaurar arquivos e os valores originais. 
Digitalis não é driver gráfico e não substitui Mesa, Vulkan, ANGLE, o driver do host ou outras partes do stack gráfico. A disponibilidade do
Native Bridge não garante que todo APK ARM64, jogo ou aplicativo Vulkan funcione. 
Instalar, alternar e remover 
# Android 11/13 x86_64
sudo wcs install libndk
sudo wcs install libhoudini
 # Imagem específica Android 16 QPR2 WayDroid-ATV
sudo wcs install digitalis
 # Remover/restaurar o provider ativo
sudo wcs remove libndk
sudo wcs remove libhoudini
sudo wcs remove digitalis
 Use apenas o comando correspondente à combinação realmente suportada. O WCS bloqueia gerações para as quais não há payload fixado
nesta versão. 
 Magisk e módulos 
Adaptador Magisk WCS 31.0 
O WCS baixa o APK Magisk 31.0 de uma URL fixada e confere o SHA-256 configurado em config/defaults.conf. Em seguida, extrai os
binários necessários, aplica patches em offsets específicos somente quando os bytes originais esperados coincidem e modifica arquivos do
overlay Waydroid. 
A integração utiliza gatilhos em system/etc/init/bootanim.rc; não é um patch de boot.img. O arquivo original de bootanim.rc é
preservado para restauração. 
Comandos: 
sudo wcs root status
sudo wcs root install --stop
sudo wcs root remove
sudo wcs root recover
 Faixa declarada pelo código: Android API 30–37, com conjuntos de patches para x86_64, x86, arm64-v8a e armeabi-v7a, quando a
arquitetura/patch correspondente existe. 
O adaptador é experimental. root status separa a presença de arquivos/registro/backup da comprovação de runtime. O probe via ADB tenta
verificar daemon, versão Magisk e su, mas não considera uma shell que já esteja como UID 0 uma prova independente de MagiskSU. 
wcs root remove restaura somente os caminhos de overlay pertencentes à integração do WCS. Módulos instalados em /data/adb/modules
e dados persistentes não são apagados por essa operação. 
Instalação de módulos de terceiros 
O WCS valida ZIPs de módulos antes de enviar o arquivo para /data/local/tmp no guest. A instalação é executada pelo Magisk dentro de
waydroid shell, recebendo o script via stdin. O WCS exige um marcador explícito de código de saída do instalador Magisk e verifica se o
módulo apareceu em modules ou modules_update. 
Comandos:sudo wcs modules list
sudo wcs modules install ./modulo.zip
sudo wcs modules enable modulo_id
sudo wcs modules disable modulo_id
sudo wcs modules remove modulo_id
 enable/disable criam/removem os marcadores do Magisk e entram em vigor no próximo boot. remove agenda a remoção para o próximo
boot; não apaga imediatamente a pasta ativa. 
 APKs e aplicativos 
wcs apps list
wcs apps install ~/Downloads/aplicativo.apk
wcs apps remove com.exemplo.aplicativo
 O WCS valida que o arquivo APK é regular e contém manifesto Android antes de solicitar a instalação através do Waydroid. A listagem e a
remoção usam a CLI do Waydroid. 
O WCS não garante compatibilidade de um APK específico com a imagem, ABI, serviços Google, GPU ou versão Android. 
 Diagnóstico, microG e certificação 
wcs extras 
Coleta informações dentro do Android em execução, incluindo: 
presença dos pacotes GSF, Google Play Services e Play Store;
pacotes relacionados ao microG;
valor atual de ro.dalvik.vm.native.bridge;
caminhos de bibliotecas Widevine localizadas;
diagnóstico de pacotes, caminhos privilegiados, allowlists, indicador de assinatura falsa, WebView/Custom Tabs e estado de GServices
quando acessíveis. 
A presença de pacotes com nomes Google não distingue sozinha GApps de microG. Da mesma forma, encontrar uma biblioteca Widevine não
prova reprodução DRM ou licença funcional. 
wcs microg 
Executa consultas somente leitura dentro do guest. É uma coleta de evidências, não um teste definitivo de autenticação. 
Para o módulo microG DresOS: 
requer Android API 26–36;
requer imagem sem GApps;
requer Magisk;
requer que a ROM ofereça suporte à assinatura falsa (signature spoofing). 
O módulo não adiciona por conta própria Zygisk/LSPosed para assinatura falsa. Após instalar e reiniciar, confira microG Settings → Self-Check,
principalmente “System spoofs signature”. Em seguida, teste registro do dispositivo, login Google, sincronização e Play Store/Aurora. O WCS
não marca autenticação como aprovada sem esses testes dentro do Android. 
wcs certified 
Tenta obter o Android ID no banco gservices.db do Google Services Framework. É necessário que o GSF esteja instalado/inicializado e que
sqlite3 exista no guest. O comando informa o endereço do Google usado para registrar dispositivos não certificados. 
 Arquivos e diretórios utilizados 
Os caminhos padrão podem ser alterados, quando aplicável, pelas variáveis de ambiente indicadas.Caminho Finalidade/var/lib/waydroid/waydroid.cfg Configuração Waydroid padrão; variável WAYDROID_CONFIG/var/lib/waydroid/images/ Caminho padrão de imagens; o valor efetivo é lido da configuração Waydroid/var/lib/waydroid/overlay//var/lib/waydroid/overlay_rw/ Camada writable identificada pela instalação Waydroid/var/lib/waydroid/data/ Fallback de dados Android quando não existe data_path configurado ou diretório de usuário
detectável/var/lib/wcs/ Diretório de estado padrão; variável WCS_STATE_DIR/var/lib/wcs/backups/images/ Backups de system.img e vendor.img/var/lib/wcs/backups/root/ Snapshots arquivados da integração Magisk e pontos de recuperação/var/lib/wcs/root-state/ Estado/backup-base da integração Magisk WCS/var/lib/wcs/arm-translation/ Estado e backups de arquivos/propriedades de Native Bridge gerenciados pelo WCS/var/lib/wcs/cache/arm-translation/ Pacotes de libndk/libhoudini baixados e validados/var/lib/wcs/cache/digitalis-qpr2/ Artefatos Digitalis QPR2 baixados e validados/var/lib/wcs/downloads/features/ Downloads de componentes como microG, Smart Dock, F-Droid Privileged e módulo CA/usr/local/lib/wcs/ Código instalado pelo install.sh no prefixo padrão/usr/local/bin/wcs Link simbólico do comando no prefixo padrãoQuando data_path não está configurado ou não aponta para um diretório existente, o WCS procura o diretório de dados do usuário
(~/.local/share/waydroid/data) e depois o fallback ao lado da configuração Waydroid. O caminho real detectado é exibido por wcs infowcs doctor. 
Para instalar em outro prefixo, use PREFIX nos scripts de instalação/desinstalação. WCS_STATE_DIR altera o estado do WCS e
WAYDROID_CONFIG altera o caminho do arquivo de configuração Waydroid. 
 Segurança e comportamento das operações 

e
O WCS aplica verificações antes de operações sensíveis. Entre elas: 
rejeição de caminhos com componentes simbólicos em operações que manipulam arquivos protegidos;
validação de nomes de pacote, IDs de módulo e caminhos internos de ZIP;
limites de tamanho e quantidade de entradas para arquivos baixados/compactados;
validação de CRC para ZIPs e SHA-256/MD5 conforme o formato e a origem do payload;
download por HTTPS a partir de hosts permitidos nos instaladores correspondentes;
arquivos temporários e publicação do download somente depois da validação;
snapshots dos arquivos do overlay antes de substituí-los;
verificação de manifesto, tamanho e SHA-256 antes da restauração de backups;
recusa de escrita quando não é possível comprovar que o Waydroid está parado;
recusa de payloads quando a versão/API/arquitetura não corresponde ao conjunto de arquivos fixado. 
A detecção pode continuar disponível com Waydroid parado. Muitas operações de escrita, instalação de componentes ou alterações do overlay
exigem estado parado; instalação/listagem de APKs e operações de módulos dependem do Android em execução. 
Não interrompa o processo durante uma gravação de imagem, substituição de overlay, instalação ou restauração. Mantenha os backups até
confirmar que a instalação/remoção funcionou no Android de destino. 
 Testes e limites conhecidos 
Testes automatizados incluídos 
Execute a suíte a partir da pasta wcs/: 
python3 -m unittest discover -s tests -v
 Na auditoria desta distribuição WCS 0.5.0, a suíte foi executada a partir do ZIP extraído: 103 testes passaram. Os testes cobrem, entre outros
pontos: 
detecção de Android/API, arquitetura, canais oficiais e estado LXC;
parsing e disponibilidade de manifestos OTA;
segurança de caminhos, links simbólicos e downloads verificados;
criação/restauração de backup e validação de manifestos;
patches e snapshots da integração Magisk;
validação e transporte de módulos Magisk pelo stdin de waydroid shell;
validação e troca dos providers libndk/libhoudini;
manifesto, hashes, arquitetura ELF, bloqueios de imagem, instalação simulada e rollback do Digitalis;
bloqueios de recursos legados sem port validado.Overlay que recebe alterações do WCS quando habilitadoO que os testes host não comprovam 
A suíte usa testes automatizados no host e fixtures/simulações para operações de imagem e overlay. Ela não inicializa uma imagem Android
real. Portanto, não comprova automaticamente: 
boot de cada imagem Waydroid;
funcionamento do daemon Magisk, MagiskSU, Zygisk, LSPosed ou Vector;
signature spoofing, login Google ou sincronização microG;
permissões efetivas do F-Droid Privileged;
confiança da CA após reinicialização;
execução de todos os APKs ARM/ARM64;
reprodução Widevine/DRM;
desempenho ou compatibilidade gráfica/Vulkan. 
Limites relevantes 
Catálogo de imagens não significa que todos os manifestos existam. Uma combinação só aparece quando o servidor publica um
manifesto válido.
Imagens pré-instaladas/customizadas não são convertidas automaticamente em canais OTA. O WCS identifica e inspeciona esses casos,
mas pode bloquear a troca.
Backups de imagem não incluem /data. Contas, apps e dados pessoais exigem backup separado.
Trocar imagem pode recriar a camada writable do overlay. A integração Magisk WCS deve ser removida antes da troca; módulos de
terceiros também podem precisar de revisão.
libndk/libhoudini: payloads desta versão estão fixados para Android 11 e 13 x86_64; não force esses binários em outras gerações.
Digitalis: bundle específico Android 16 QPR2; não usar em Android 16 genérico/QPR0, Android 17 ou outra ROM.
Magisk: os arquivos podem ser preparados sem que root/runtime funcione após o boot. Confirme com wcs root status e teste
adequado via ADB.
Componentes experimentais: F-Droid Privileged e CA exigem reinicialização e verificação real.
Recursos legados: nodataperm e hidestatusbar permanecem bloqueados; o WCS não aplica patches indiscriminadamente em
services.jar ou overlays antigos.
Zygisk/LSPosed/Vector: o WCS pode relatar compatibilidade/evidências, mas não instala nem certifica esses frameworks
automaticamente. 
A matriz detalhada está em docs/COMPATIBILITY.md. O plano de testes está em docs/TEST-PLAN.md. A comparação dos recursos herdados
está em docs/LEGACY-PARITY.md, e as notas de pesquisa em docs/RESEARCH.md. 
 Estrutura do projeto 
wcs/
├── bin/├── config/
│ ├── defaults.conf│ ├── android16.conf│ └── android17.conf├── core/
│ ├── cli.py│ ├── common.py│ ├── detection.py│ ├── image_manager.py│ ├── backup.py│ ├── root_manager.py│ ├── module_manager.py│ ├── feature_manager.py│ ├── arm_translation.py│ ├── digitalis_translation.py│ ├── extras.py│ ├── compatibility.py│ ├── doctor.py│ └── recovery.py├── docs/
│ ├── COMPATIBILITY.md│ ├── LEGACY-PARITY.md│ ├── RESEARCH.md│ └── TEST-PLAN.md├── modules/├── root/magisk/# reservado para ferramentas auxiliares
# versão, caminhos padrão, fontes e metadados fixados
# metadados do perfil Android 16 QPR2
# metadados do perfil Android 17 QPR0
# CLI e menu interativo
# caminhos, downloads, checksums e execução de comandos
# detecção Waydroid/Android/ABI/OTA/overlay
# catálogo OTA, ATV e verificação local
# backup/restauração de system.img e vendor.img
# integração Magisk WCS
# ciclo de vida de módulos Magisk
# catálogo e instaladores de componentes
# libndk/libhoudini
# Digitalis/Berberis QPR2
# APKs, microG, Android ID, GApps/Widevine/bridge
# relatório de compatibilidade declarada
# verificações de host e Waydroid
# pontos de recuperação
# matriz de compatibilidade
# recursos herdados e limitações
# notas de pesquisa e decisões técnicas
# plano e critérios de teste
# documentação/reserva para assets de módulos
# documentação/reserva de assets Android do WCS├──├──├──├──├──├──├──├──├──├──└──
tests/third_party/utils/install.shuninstall.shwcswcs.shLICENSETHIRD_PARTY_NOTICES.mdCHANGELOG.mdREADME.md# testes unitários e de integração host-side
# notas de licença/proveniência de bundles de terceiros
# espaço reservado para utilitários
# instalação em PREFIX/lib/wcs e PREFIX/bin/wcs
# remoção do código instalado
# entrypoint Python
# wrapper Bash
# GPL-3.0-or-later
# avisos de componentes de terceiros
# histórico de versões
# documentação principal
 Licenças e créditos 
O código do WCS é distribuído sob GNU General Public License v3.0 or later. Consulte LICENSE. 
O projeto também utiliza ou integra artefatos de terceiros em funções específicas. Esses componentes mantêm suas próprias licenças, termos
e condições; a licença GPL do WCS não substitui as licenças dos binários baixados. Consulte THIRD_PARTY_NOTICES.md e third_party
DIGITALIS-BUNDLE-NOTICE.md. 
Referências upstream usadas pelo código incluem: 
Waydroid — contêiner Android, CLI e inicializador de imagens.
Waydroid Script — referência para recursos herdados e payloads libndk/libhoudini.
DresOS Magisk Modules — módulo microG usado pelo instalador.
F-Droid — Smart Dock e OTA da extensão privilegiada.
Magisk — APK upstream fixado para o adaptador WCS.
Refract / Digitalis QPR2 bundle — bundle Digitalis/Berberis específico integrado pelo WCS.
AOSP Binary Translation / Berberis — base aberta do mecanismo Berberis. 
WCS não reivindica autoria dos componentes de terceiros nem concede direitos adicionais sobre seus binários
