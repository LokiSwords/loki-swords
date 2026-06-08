<div align="center">

<img src="https://raw.githubusercontent.com/WydPlataform/wyd-desktop/main/resources/icon.png" width="80" height="80" alt="Loki Swords" />

# Loki Swords

**Cliente de automação desktop para MMORPGs**

[![Latest Release](https://img.shields.io/github/v/release/WydPlataform/loki-swords?style=flat-square&label=versão&color=6366f1)](https://github.com/WydPlataform/loki-swords/releases/latest)
[![Platform](https://img.shields.io/badge/plataforma-Windows-blue?style=flat-square)](https://github.com/WydPlataform/loki-swords/releases/latest)
[![License](https://img.shields.io/badge/licença-proprietária-red?style=flat-square)](#licença)

</div>

---

## O que é o Loki Swords?

Loki Swords é um cliente desktop de automação para MMORPGs. Com ele você cria macros visuais, configura rotas de navegação e deixa o personagem trabalhando enquanto você faz outras coisas — sem abrir terminais, sem editar arquivos de configuração.

Tudo é feito pelo próprio app: você define o que o personagem deve fazer, salva e aperta play.

---

## Funcionalidades

### Macros visuais
Monte fluxos de automação com blocos de execução, condições de saída e ramificações. Cada bloco define ações do personagem — cliques, teclas, esperas — e quando passar para o próximo bloco.

### Automação Dragon Field
Retorno automático ao Dragon Field após morte ou saída. O personagem detecta a posição atual, traça a rota e volta ao spot configurado. Suporta separação de lógica por tipo de personagem (Físico / Mago).

### Blocos paralelos (event-driven)
Defina reações a eventos em tempo real: ao detectar a tela de morte, ao encontrar uma imagem na tela, ao sair de uma região. Esses blocos rodam em paralelo com o fluxo principal.

### Leitor de coordenadas ao vivo
Visualize as coordenadas do personagem em tempo real direto no app — sem precisar alternar entre janelas.

### Macros com variáveis
Use `{{destX}}`, `{{destY}}` e outras variáveis nas condições e ações dos macros. Os valores são resolvidos da sua configuração local, então cada conta tem seus próprios parâmetros.

### Múltiplas instâncias
Selecione qual instância do jogo o agent deve controlar quando mais de uma estiver aberta.

---

## Download e instalação

1. Vá para a aba **[Releases](https://github.com/WydPlataform/loki-swords/releases/latest)**
2. Baixe o arquivo `Loki.Swords.Setup.X.X.X.exe`
3. Execute o instalador com permissão de administrador
4. Abra o app e entre com sua licença

> O instalador requer permissão de administrador para leitura de memória do processo do jogo.

### Atualizações automáticas

O Loki Swords verifica por atualizações ao abrir. Quando uma nova versão estiver disponível, você será notificado e poderá instalar com um clique — sem precisar baixar nada manualmente.

---

## Primeiros passos

### 1. Login
Entre com o **token de licença** e a **senha** fornecidos pelo administrador.

### 2. Configurar o macro Dragon Field
- Vá em **Macros** → selecione o macro Dragon Field
- Em **Endereços de Memória**, insira os endereços X e Y do personagem (obtidos via Cheat Engine)
- Em **Spot Final**, insira as coordenadas de destino
- Selecione o tipo de personagem (**Físico** ou **Mago**)
- Salve a configuração

### 3. Iniciar
- Selecione a instância do jogo na lista
- Clique em **Iniciar Agent**
- Acompanhe os logs em tempo real no painel lateral

---

## Requisitos

| Requisito | Versão mínima |
|-----------|--------------|
| Windows   | 10 (64-bit)  |
| Python    | 3.10+        |
| RAM       | 4 GB         |

> Python precisa estar instalado e disponível no PATH do sistema.

---

## Distribuição de licenças

O Loki Swords é um software proprietário distribuído por licença. Para adquirir uma licença, entre em contato com o administrador do sistema.

Cada licença define:
- Número máximo de instâncias simultâneas
- Período de validade
- Permissões de acesso

---

## Licença

Software proprietário — todos os direitos reservados.  
Este software não pode ser redistribuído, modificado ou utilizado sem licença válida.
