<div align="center">

<img src="https://raw.githubusercontent.com/LokiSwords/wyd-desktop/main/resources/icon.png" width="80" height="80" alt="Loki Swords" />

# Loki Swords

**Cliente de automação desktop para MMORPGs**

[![Latest Release](https://img.shields.io/github/v/release/LokiSwords/loki-swords?style=flat-square&label=versão&color=6366f1)](https://github.com/LokiSwords/loki-swords/releases/latest)
[![Platform](https://img.shields.io/badge/plataforma-Windows-blue?style=flat-square)](https://github.com/LokiSwords/loki-swords/releases/latest)
[![License](https://img.shields.io/badge/licença-proprietária-red?style=flat-square)](#licença)

</div>

---

## O que é o Loki Swords?

Loki Swords é um cliente desktop de automação para MMORPGs. Com ele você deixa o personagem trabalhando enquanto faz outras coisas — sem abrir terminais, sem editar arquivos de configuração.

Tudo é feito pelo próprio app: você configura uma vez e aperta play.

---

## Funcionalidades

### Automação Dragon Field
Retorno automático ao Dragon Field após morte ou saída. O personagem detecta a posição atual, traça a rota e volta ao spot configurado automaticamente. Suporta Físico e Mago.

### Múltiplas instâncias
Controle mais de uma conta ao mesmo tempo. Selecione qual instância do jogo cada agent deve controlar.

### Atualizações automáticas
O app verifica por atualizações ao abrir e instala com um clique — sem precisar baixar nada manualmente.

---

## Download e instalação

1. Vá para a aba **[Releases](https://github.com/LokiSwords/loki-swords/releases/latest)**
2. Baixe o arquivo `Loki.Swords.Setup.X.X.X.exe`
3. Execute o instalador com permissão de administrador
4. Abra o app e entre com sua licença

> O instalador requer permissão de administrador para leitura de memória do processo do jogo.

---

## Primeiros passos

### 1. Login
Entre com o **token de licença** e a **senha** fornecidos pelo administrador.

### 2. Configurar o Dragon Field
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

## Licenças

O Loki Swords é distribuído por licença. Para adquirir uma licença, entre em contato com o administrador do sistema.

Cada licença define o número máximo de instâncias simultâneas e o período de validade.

---

## Aviso legal

Software proprietário — todos os direitos reservados.  
Este software não pode ser redistribuído, modificado ou utilizado sem licença válida.
