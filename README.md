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

As informações a seguir apresentam a estrutura completa da suíte de testes do qaFood, organizada em cinco Features que representam, em sequência, a jornada do usuário no aplicativo. Cada Feature possui um conjunto de subtópicos que agrupa os cenários por funcionalidade e nível crescente de complexidade, permitindo visualizar de forma clara o que é validado em cada etapa, desde o login até a finalização e o acompanhamento do pedido

| Feature | O que valida | Papel na jornada |
|---|---|---|
|**1.Login** | Autenticação, campos, erros de validação, comportamento do botão de acesso | Ponto de entrada — "quero acessar o app" |
|**2.Lojas** | Listagem, busca, navegação e permissão de localização | "Onde quero pedir?" |
|**3.Cardápio** | Produtos, carrinho, contador, navegação dentro do restaurante | "O que vou comer?"|
|**4.Sacola** | Gerenciamento do carrinho, subtotal, persistência | "Revisar minha compra" |
|**5.Pedido** | Confirmação, pagamento, finalização, acompanhamento | "Confirmar, pagar e receber" |

### 1. Feature Login
Valida o processo de autenticação e o comportamento dos campos e botão de acesso, organizado em 8 subtópicos:

#1. Fluxo básico — login com credenciais corretas, campos vazios (e-mail, senha, ou ambos) e bloqueio de envio correspondente.
#2. Validação de formato e conteúdo — e-mail não cadastrado, espaços em branco isolados ou combinados, espaços nas pontas, maiúsculas, e caracteres especiais (+, apóstrofo).
#3. Validação de senha — senha incorreta, maiúsculas na senha, e limites de tamanho (e-mail e senha muito longos).
#4. Correção e recuperação de erro — corrigir o e-mail antes do envio, e corrigir a senha após um erro até conseguir entrar.
#5. Interações com teclado e sistema operacional — tecla Enter/Done, rotação de tela durante o preenchimento, e retorno do app após ida para segundo plano.
#6. Cliques repetidos e comportamento de interface — duplo clique sequencial, cliques fixos, e cliques repetidos até erro (com e sem confirmação).
#7. Concorrência e condição de corrida — duplo toque simultâneo no botão Entrar.
#8. Bloqueio por tentativas de senha — mesma conta e contas diferentes com senha errada em sequência, e reforço do teste de campo vazio.

### 2. Feature Lojas
Valida exibição, navegação e pesquisa dos restaurantes, organizada em 8 subtópicos:

#1. Acesso básico à tela de Lojas — login com acesso à lista, abertura de cardápio, e scroll inicial.
#2. Navegação e visualização da lista — scroll até cada restaurante individualmente e até o fim da lista.
#3. Permissão e seleção de endereço — abertura do modal, permitir, cancelar, e preenchimento automático do endereço.
#4. Busca básica — busca por caractere único, termo parcial, restaurante inexistente, e limpeza da busca para nova pesquisa.
#5. Busca por restaurantes específicos — busca pelo nome exato de cada um dos 6 restaurantes cadastrados.
#6. Busca com espaços e capitalização — espaços nas pontas, e a matriz completa de maiúsculas/minúsculas (total, parcial por palavra, e mista).
#7. Casos de borda da busca — apenas espaços em branco, caracteres especiais/números isolados ou misturados com nome válido.
#8. Persistência e ciclo de vida do aplicativo — sessão perdida ao fechar/reabrir o app, e re-login funcional na sequência.

### 3. Feature Cardápio
Valida o acesso aos restaurantes e o comportamento dos produtos, organizada em 7 subtópicos:

#1. Acesso e carregamento básico do cardápio — acesso a diferentes restaurantes, bloqueio sem endereço selecionado, e permissão de localização não solicitada novamente.
#2. Validação dos elementos do cardápio — cabeçalho do restaurante, nome/preço/descrição do item, e carrinho vazio ao abrir.
#3. Navegação dentro e fora do cardápio — scroll para baixo e para cima, e retorno à tela anterior (via botão da UI e via botão físico Voltar).
#4. Adição de um produto ao carrinho — adicionar um item, confirmar contador, e persistência ao sair da página.
#5. Adição e persistência de múltiplos produtos — adicionar vários itens, persistência de todos ao sair, e ausência de duplicação/perda após idas e vindas.
#6. Persistência do estado em diferentes condições — contador mantido após rotação de tela.
#7. Cenário de concorrência / múltiplas ações rápidas — duplo toque rápido no botão de adicionar.

### 4. Feature Sacola (Carrinho)
Valida o funcionamento do carrinho, organizada em 7 subtópicos:

#1. Operações básicas da Sacola — abrir vazia, abrir após adicionar, adicionar e remover, e adicionar o mesmo item duas vezes.
#2. Cálculo/subtotal — soma dos itens adicionados, e soma quando o mesmo item é duplicado.
#3. Navegação entre Sacola e Cardápio — retorno ao cardápio preservando o item, e adição de um segundo produto diferente após o retorno.
#4. Cancelamento — desistir da limpeza da sacola, e desistir da remoção de um item.
#5. Limpeza da Sacola — limpar com produtos diferentes, botão Limpar habilitado mesmo vazio, e readicionar item via botão "Adicionar itens".
#6. Múltiplos produtos e preservação de estado — três itens diferentes com subtotal correto, e botão físico Voltar preservando os itens.
#7. Comportamento do aplicativo — rotação de tela, perda de sessão de login, e perda do conteúdo da sacola ao fechar/reabrir o app.

### 5. Feature Pedido
Valida a confirmação e finalização do pedido, organizada em 7 subtópicos:

#1. Acesso e confirmação básica do pedido — subtotal/taxa/total com um item, e soma correta com múltiplos itens.
#2. Validação de dados e condições obrigatórias — alerta sem forma de pagamento selecionada, cupom vazio, e cupom inválido.
#3. Cancelamento da finalização e preservação do carrinho — voltar da tela de confirmação sem finalizar, mantendo o carrinho intacto.
#4. Realização do pedido por diferentes formas de pagamento — pedido com Dinheiro e com Cartão de crédito, incluindo confirmação de sucesso.
#5. Validação completa do pedido realizado — conferência de todos os dados da tela de acompanhamento (status, previsão, endereço, pagamento, total).
#6. Navegação após a conclusão do pedido — retorno à página de Lojas após finalizar.
#7. Persistência do estado após alteração de orientação — rotação de tela na tela de acompanhamento pós-conclusão.

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
