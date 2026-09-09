# qaFood — Testes Automatizados com Maestro

Suíte de testes end-to-end para o aplicativo **qaFood**, uma versão do **iFood** utilizada como projeto de estudo, desenvolvida pela escola **Qazando** (professores Eduardo Finotti e Hebert Soares).

Todos os testes e a estrutura deste repositório foram criados por **Diogo Amancio**, com base nos conhecimentos adquiridos no curso **Automação Mobile com Maestro**, utilizando Android Studio, WSL (Linux) e Maestro.

---

## 📱 Sobre o app

O qaFood simula um aplicativo de delivery completo (iFood), cobrindo a jornada real de um usuário: login, busca de restaurantes, navegação por cardápio, carrinho de compras e finalização de pedido.

<img width="405" height="860" alt="image" src="https://github.com/user-attachments/assets/15ad61e5-2117-42e6-a2d8-b7ddb1f90092" />


---

## 🛠️ Ambiente e rotina diária

O ambiente de testes combina **emulador Android no Windows** + **WSL (Linux)** + **Maestro CLI/Studio**.

### 1. Abrir o emulador (PowerShell)
```powershell
cd $env:LOCALAPPDATA\Android\Sdk\emulator
.\emulator.exe -avd Pixel_4 -gpu swiftshader_indirect
```
Aguarde o emulador carregar completamente antes de seguir. **Não** use o botão ▶ do Android Studio — sempre use esse comando.


### 2. Instalação do app (qafoodcompletao.apk)
```powershell
cd $env:LOCALAPPDATA\Android\Sdk\emulator
.\emulator.exe -avd Pixel_4 -gpu swiftshader_indirect
```
Aguarde o emulador carregar completamente antes de seguir. **Não** use o botão ▶ do Android Studio — sempre use esse comando.
O arquivo .apk é apenas o **instalador do aplicativo.** Ele **não faz parte da conexão entre Windows, WSL, ADB e Maestro.**

A comunicação dos testes depende apenas de:
**Emulador rodando → ADB conectado via rede → Maestro apontando para o host correto**

**O APK só precisa ser instalado nos seguintes casos:**
1. Emulador novo, sem o app instalado;
2. Reset/Wipe do emulador, que remove os aplicativos instalados;
3. Necessidade de reinstalar o aplicativo;
4. Necessidade de trocar a versão do aplicativo.

**Para instalar o APK:**

Confirme que o emulador está rodando e conectado:
```bash
adb devices
```
Instale o APK no device correto:

```bash
adb -s <IP>:25555 install caminho/para/qafoodcompletao.apk
```

Confirme que a instalação foi realizada:

```bash
adb shell pm list packages | grep qazandoqafood
```

O resultado esperado é:

```bash
package:com.qazandoqafood
```

📌 **Importante: depois que o aplicativo estiver instalado, o arquivo .apk não precisa ser utilizado novamente para executar os testes. O Maestro interage diretamente com o aplicativo por meio do appId: com.qazandoqafood.**

### 3. Conectar o WSL ao emulador
```bash
adb kill-server
adb connect <IP>:25555
adb devices
```
> ⚠️ O IP não é fixo — ele pode mudar a cada reinício do Windows/WSL. Descubra o valor atual com:
> ```bash
> ip route show default | awk '{print $3}'
> ```
> Resultado esperado do `adb devices`: `<IP>:25555   device`

### 4. Executar os testes com Maestro
```bash
maestro --host <IP> test <caminho-do-arquivo>.yaml
```

### 5. Abrir o Maestro Studio (interface visual, opcional)
```bash
cd ~/Downloads
./MaestroStudio.AppImage
```
Selecione o device `<IP>:25555` na lista.

> ⚠️ O streaming de tela ao vivo dentro do Studio não funciona neste ambiente (erro de gRPC pela rede) — os testes rodam normalmente mesmo assim. Para acompanhar visualmente, use a janela do emulador no Windows.

**Ordem diária:** PowerShell (emulador) → WSL/ADB (conexão) → Maestro/Maestro Studio (execução)

---

## 📁 Estrutura do repositório

```
Maestro/
├── 1 - Feature_Login/
├── 2 - Feature_Lojas/
├── 3 - Feature_Cardápio/
├── 4 - Feature_Sacola (Carrinho)/
└── 5 - Feature_Pedido/
```

Cada teste que depende de login reutiliza o mesmo flow base via `runFlow`, evitando duplicação:
```yaml
- runFlow:
    file: ../1 - Feature_Login/20 - Login com credenciais corretas.yaml
```
> 📌 O caminho do `runFlow` é sempre **relativo ao arquivo que o chama**. Testes salvos dentro de uma subpasta de feature usam `../1 - Feature_Login/...` (sobem um nível antes de entrar em `1 - Feature_Login`). Testes salvos direto na raiz `Maestro/` usam o caminho sem `../`.

---

## 🧭 A jornada do usuário e as 5 Features

A suíte tem como objetivo automatizar e validar a jornada completa do usuário dentro do qaFood:

```
Login → Lojas → Cardápio → Sacola → Pedido → Acompanhamento
```
Dessa forma, os testes não validam apenas funcionalidades isoladas, mas também simulam comportamentos e situações próximas da utilização real de um aplicativo de delivery.


| Feature | O que valida | Papel na jornada |
|---|---|---|
|**1.Login** | Autenticação, campos, erros de validação, comportamento do botão de acesso | Ponto de entrada — "quero acessar o app" |
|**2.Lojas** | Listagem, busca, navegação e permissão de localização | "Onde quero pedir?" |
|**3.Cardápio** | Produtos, carrinho, contador, navegação dentro do restaurante | "O que vou comer?"|
|**4.Sacola** | Gerenciamento do carrinho, subtotal, persistência | "Revisar minha compra" |
|**5.Pedido** | Confirmação, pagamento, finalização, acompanhamento | "Confirmar, pagar e receber" |

### 1. Feature Login
Valida o processo de autenticação e o comportamento dos campos e botão de acesso: campos vazios, credenciais inválidas, formatos de e-mail, sensibilidade a maiúsculas/minúsculas, espaços em branco, limites de caracteres, caminho feliz, recuperação de erro, cliques múltiplos/duplo toque, tentativas repetidas de senha incorreta, e interações com o sistema operacional (Enter, Home, background, rotação de tela).

### 2. Feature Lojas
Valida exibição, navegação e pesquisa dos restaurantes: acesso à lista após login, scroll, localização de restaurantes específicos, busca por nome completo/parcial/inexistente, espaços em branco, caracteres especiais, sensibilidade a maiúsculas/minúsculas, buscas consecutivas, permissão de localização (aceitar/recusar/não solicitar novamente), e persistência de sessão após fechar/reabrir o app.

### 3. Feature Cardápio
Valida o acesso aos restaurantes e o comportamento dos produtos: acesso ao cardápio de diferentes restaurantes, bloqueio sem endereço selecionado, retorno à tela anterior, adição de um ou vários produtos, contador de produtos, produtos diferentes e duplicados, nome/preço/descrição, scroll, persistência de itens após navegação, manutenção do contador após rotação de tela, duplo toque rápido, e cabeçalho do restaurante.

### 4. Feature Sacola (Carrinho)
Valida o funcionamento do carrinho: abrir com/sem produtos, adicionar o mesmo produto múltiplas vezes, quantidade e preço, adicionar/remover, confirmar/cancelar limpeza, produtos diferentes, retorno ao cardápio sem perder itens, botão Limpar com carrinho vazio, readicionar itens, rotação de tela, e cálculo de subtotal em diferentes combinações.

### 5. Feature Pedido
Valida a confirmação e finalização do pedido: cupom inválido/vazio, subtotal/taxa de entrega/total, produtos no pedido, formas de pagamento (cartão de crédito, dinheiro), alerta ao tentar finalizar sem forma de pagamento, tela de "Pedido realizado" (previsão de entrega, status, endereço, detalhes, pagamento, total), retorno à tela de Lojas, rotação de tela pós-conclusão, e retorno da confirmação sem finalizar (carrinho permanece intacto).

---

## 💡 Aprendizados técnicos 

- **Sintaxe YAML**: `appId` fica no cabeçalho do arquivo (antes do `---`), nunca dentro da lista de comandos. Seletores como `id:` precisam de indentação correta e espaço após os dois-pontos (`id: "email"`, não `id:"email"`).
- **`tapOn` não digita** — sempre seguido de `inputText` para preencher campos.
- **Ambiguidade de seletores**: `rightOf: "texto"` e `point: "x%,y%"` são frágeis após scroll (podem clicar no elemento errado sem gerar erro). Prefira `id` real do elemento — descoberto via `maestro hierarchy` ou pelo Inspector do Maestro Studio.
- **`scrollUntilVisible` é just-in-time**: revelar um elemento não garante que os próximos também fiquem visíveis. Use um `scrollUntilVisible` por elemento que precisa ser tocado/validado.
- **Alertas e pop-ups de confirmação**: título e corpo do modal geralmente são apenas `assertVisible` (informativos); só o botão de ação final é `tapOn`.
- **IDs confirmados no app**: `add-item-buttom` (sic — contém erro de digitação no próprio app), `open-cart-button`, `back-button`.
- **Mensagens reais confirmadas**: `"Erro ao realizar login"` (erro genérico de autenticação), `"CUPOM inválido"`, `"Selecione uma forma de pagamento"`.
- Fechar/reabrir o app com `launchApp: clearState: false` **não preserva a sessão de login** neste app — é necessário refazer o `runFlow` de login mesmo sem limpar o estado.
