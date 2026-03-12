As seguintes alteraçòes serão necessárias. Separei por item.

### 3. Pagamentos e Assinatura (BC4, BC6)

a) separar em 2 itens distintos: a) Assintaura da plataforma Loja integrada, diz respeito ao fluxo de visitar a pagina de planos, selecionar um dos planos (Crescimento, Aceleraçào, Expansão Elite), iniciar o checkout e finalizar a assinatura da plataforma.

b) Fluxo de ativação dos gateway de pagamento: Pagali, nesse fluxo compreender:

1. **Criação de conta**
   - Acesso ao Pagali
   - Preenchimento de dados básicos

2. **Verificação de conta**
   - Envio de documentação
   - Validação de dados
   - Ativação de funcionalidades completas
   - Aumento de limite de saque

3. **Configuração de meios de pagamento**
   - Ativação de Pix
   - Ativação de cartão de crédito
   - Ativação de boleto
   - Configuração de link de pagamento

Sobre nomeclatura dos eventos desse fluxo, entendo que existem outros gateways de pagamento, como por exemplo o Mercado Pago, App Max, PagSeguro, pagar.me, Paypal e Pagali.

Sendo assim gostaria que o fluxo se mantivesse generico para ativação de qualquer um desses gateways de pagamento e incluissemos uma propriedade gateway que leva o valor do gateway escolhido.

Para realizar as alterações garanta que os Business cases continuam factiveis.

Verifique todos os arquivos

Crie um plano detalhado, o objetivo dessa alteração é simplificar a inserção de eventos e a atualizaçào posterior.

---

### 6. Canais de Venda (BC11)

Eu tenho varios markeplaces passiveis de ativação (Magalu, Mercado Livre, Allever, Compre Sua Peça), então gostaria que o fluxo mapeado fosse "genérico" e com isso englobasse todos e para separar utilizassemos uma propriedade de evento marketplace.

Então ao inves de ML connection, seria algo como MarketPlace Connection Started com uma propriedade marketplace = "Mercado Livre"

Para realizar as alterações garanta que os Business cases continuam factiveis.

Verifique todos os arquivos

Crie um plano detalhado, o objetivo dessa alteração é simplificar a inserção de eventos e a atualizaçào posterior.

---

Verifique o arquivo 02 user profile para garantirmos que os business cases são factiveis.

Essas propriedades majoritariamente devera vir do lake da Loja integrada, e a fonte principal é o banco de dados da aplicação.

Para situações onde necessario maior velocidade da informações podemos atualizar as propriedade os do usuário via front end (necessario ser pripriedade derivada de um evento), porém quero limitar essa estratégia a apenas dados de Contato e assinatura da plataforma. Quanto menos utilizar o front pra atualizar propriedades do usuário melhor. o MOtivo dessa limitação e diretriz é:

Nós temos diversos usuários na mesma loja e as funcionalidade e fluxos desenhados são a nível de loja: O que significa isso: Vamos supor que eu tenha três usuários distintos na loja, inicialmente eles estão no plano gratuito, e um dos usuários vai efetuar assinatura da loja. Do ponto de vista de reguas de comunicação já atualizamos a propriedade desse usuário, logo para esse usuário, ele já vai ser tratado como "loja com plano pago" naquele momento, porque a gente está atualizando a propriedade daquele usuário no Front-End.

Só que para os outros dois usuários, até que essa informação vá ao banco de dados, seja levada para o lake e seja enviada na plataforma (+- 2 a 8 horas depois) alterada na plataforma, os outras 2 usuários usuários vão ser considerados em comunicaçòes como "loja gratuita".

Então eu gostaria de evitar ao máximo a atualização no Front-End de propriedades do usuário, para que esses dados sejam todos atualizados via Back-End e comunicação seja coerente entre os usuários..

MAis informações sobre atualizaçào do perfil o arquivo 06.
