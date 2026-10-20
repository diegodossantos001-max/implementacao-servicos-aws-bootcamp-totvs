# RELATÓRIO DE IMPLEMENTAÇÃO DE SERVIÇOS AWS

Data: 20/10/2026
Empresa: Abstergo Industries
Responsável: Diego dos Santos

## Introdução
Este relatório apresenta o processo de implementação de ferramentas na empresa Abstergo Industries, realizado por Diego dos Santos. O objetivo do projeto foi elencar 3 serviços AWS, com a finalidade de realizar diminuição de custos imediatos.

O contexto do projeto é a criação da plataforma virtual da **Farmácia Abstergo**, uma farmácia fictícia que vende medicamentos e produtos de higiene e beleza pela internet. A plataforma precisa exibir um catálogo de produtos, receber pedidos, guardar receitas médicas enviadas pelos clientes e emitir notas e comprovantes. A demanda varia muito ao longo do dia e do mês (picos em promoções, no início do mês e em períodos de gripe), então a infraestrutura não pode ficar superdimensionada o tempo todo. Por isso, os serviços foram escolhidos pelo critério de **pagar apenas pelo que é usado**.

## Descrição do Projeto
O projeto de implementação de ferramentas foi dividido em 3 etapas, cada uma com seus objetivos específicos. A seguir, serão descritas as etapas do projeto:

Etapa 1:
- **Amazon EC2 (com Auto Scaling e Savings Plans)**
- **Foco:** computação elástica para hospedar a aplicação web da farmácia (loja virtual, carrinho e checkout), ajustando a capacidade à demanda real.
- **Caso de uso:** a aplicação roda em instâncias EC2 dentro de um grupo de Auto Scaling, atrás de um Application Load Balancer. Fora do horário comercial e de madrugada, o grupo fica com o mínimo de instâncias. Durante promoções, como a "Semana da Saúde", novas instâncias são criadas automaticamente quando o uso de CPU passa de 70% e removidas quando o pico acaba. A capacidade base, que fica ligada o tempo todo, é coberta por um **Compute Savings Plan** de 1 ano, com desconto de até 66% em relação ao preço sob demanda. Assim, a empresa deixa de pagar por servidores ociosos que antes ficavam ligados para aguentar o pior cenário.

Etapa 2:
- **Amazon S3 (com S3 Intelligent-Tiering e regras de ciclo de vida)**
- **Foco:** armazenamento de objetos barato, durável e escalável para arquivos da plataforma.
- **Caso de uso:** o S3 guarda as imagens do catálogo de produtos, as receitas médicas digitalizadas enviadas pelos clientes e as notas fiscais em PDF. Receitas e notas são muito consultadas nos primeiros dias e quase nunca depois, então ficam na classe **S3 Intelligent-Tiering**, que move os objetos sozinha para camadas mais baratas quando deixam de ser acessados. Uma regra de ciclo de vida leva os documentos com mais de 1 ano para o **S3 Glacier Deep Archive**, mantendo o histórico exigido pela legislação sanitária e fiscal a uma fração do custo. Os buckets usam criptografia (SSE-KMS) e bloqueio de acesso público, porque receitas médicas são dados pessoais sensíveis protegidos pela LGPD.

Etapa 3:
- **AWS Lambda**
- **Foco:** computação serverless, sem servidor, para tarefas que acontecem por evento e não precisam de um servidor ligado o tempo todo.
- **Caso de uso:** funções Lambda executam as rotinas de segundo plano da farmácia. Quando um cliente envia uma receita para o S3, uma função é disparada para validar o arquivo, gerar uma miniatura e avisar o farmacêutico responsável. Quando um pedido é confirmado, outra função envia o e-mail de confirmação e atualiza o estoque. Uma função agendada pelo Amazon EventBridge verifica todas as noites os produtos com estoque baixo ou perto do vencimento. A cobrança é feita por requisição e por milissegundo de execução, e o nível gratuito inclui 1 milhão de requisições por mês, então essas rotinas custam quase nada em comparação com uma instância dedicada a elas.

## Conclusão
A implementação de ferramentas na empresa *Abstergo Industries tem como esperado a redução imediata dos custos de infraestrutura, eliminando o pagamento por servidores ociosos (EC2 com Auto Scaling e Savings Plans), por armazenamento caro de arquivos pouco acessados (S3 Intelligent-Tiering e Glacier) e por máquinas dedicadas a tarefas eventuais (AWS Lambda), além de maior escalabilidade e disponibilidade da plataforma da Farmácia Abstergo em períodos de pico*, o que aumentará a eficiência e a produtividade da empresa. Recomenda-se a continuidade da utilização das ferramentas implementadas e a busca por novas tecnologias que possam melhorar ainda mais os processos da empresa. Como próximos passos, sugere-se configurar o **AWS Budgets** e o **AWS Cost Explorer** para acompanhar os gastos mensais e emitir alertas, e avaliar o **Amazon RDS** ou o **Amazon DynamoDB** para o banco de dados de produtos e pedidos.

## Anexos

- Diagrama da arquitetura da plataforma (Application Load Balancer → EC2 Auto Scaling → S3 / Lambda)
- Planilha de estimativa de custos gerada na [AWS Pricing Calculator](https://calculator.aws/)
- Documentação oficial do [Amazon EC2 Auto Scaling](https://docs.aws.amazon.com/autoscaling/)
- Documentação oficial do [Savings Plans](https://docs.aws.amazon.com/savingsplans/)
- Documentação oficial das [classes de armazenamento do Amazon S3](https://aws.amazon.com/s3/storage-classes/)
- Documentação oficial do [AWS Lambda](https://docs.aws.amazon.com/lambda/)

Assinatura do Responsável pelo Projeto:

Diego dos Santos
