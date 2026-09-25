# Descoberta de Injeção de HTML no Currículo Lattes

## Resumo

Esta documentação registra uma vulnerabilidade de **injeção de HTML** identificada na plataforma Currículo Lattes, mantida pelo CNPq.

A vulnerabilidade foi descoberta por acaso, durante o uso normal da plataforma. Após criar minha conta e acessar o currículo, observei o campo de **resumo** e inicialmente pensei apenas em personalizar sua apresentação.

Como primeiro teste, inseri uma tag HTML simples, como um `<h1>`, para verificar se era possível modificar a formatação do texto. O resultado mostrou que determinadas tags HTML eram interpretadas e renderizadas pela página.

A partir dessa observação surgiu a hipótese de que o campo poderia apresentar algum problema de segurança relacionado à injeção de HTML ou XSS. Foram realizados testes controlados para verificar essa possibilidade.

Foi confirmado que é possível inserir e renderizar determinados elementos HTML, incluindo um `<iframe>` apontando para conteúdo externo.

A vulnerabilidade já foi **reportada ao suporte responsável pela plataforma**, acompanhada da documentação e das evidências disponíveis.

## Como a descoberta aconteceu

A descoberta não começou como um teste de segurança planejado.

Após criar minha conta no Currículo Lattes, encontrei o campo de resumo do currículo e pensei em utilizar HTML para personalizar sua apresentação.

O primeiro teste foi simplesmente inserir uma tag:

```html
<h1>Teste</h1>
```

A tag foi interpretada pelo sistema e renderizada como um título.

Isso levantou uma questão:

> Se HTML está sendo interpretado nesse campo, quais elementos HTML são aceitos pelo processamento da plataforma?

A partir daí foram realizados testes controlados para entender o comportamento do mecanismo de filtragem.

## Resultado dos testes

Foi constatado que a plataforma não permite simplesmente qualquer tipo de conteúdo.

Em particular, durante os testes realizados:

* tags `<script>` não foram executadas;
* tentativas de executar JavaScript diretamente não apresentaram execução;
* manipuladores de eventos HTML testados não apresentaram execução;
* não foi demonstrado JavaScript executando no contexto do domínio do CNPq;
* determinados elementos HTML, entretanto, continuam sendo processados e renderizados;
* entre eles, foi possível inserir um `<iframe>`.

Exemplo utilizado durante a demonstração:

```html
<iframe src='https://example.com'></iframe>
```

O elemento foi posteriormente encontrado no HTML renderizado da página.

## Sobre XSS

Inicialmente, a possibilidade levantada foi a existência de uma vulnerabilidade de **Cross-Site Scripting (XSS)**, justamente porque HTML fornecido pelo usuário estava sendo interpretado pelo navegador.

Entretanto, os testes realizados não demonstraram execução de JavaScript.

Por esse motivo, a evidência obtida é mais precisamente descrita como uma **injeção de HTML com possibilidade de incorporação de conteúdo externo**, enquanto a classificação definitiva de severidade e impacto deve ser determinada pelo responsável pela plataforma.

O fato de o mecanismo impedir `<script>` e outras tentativas de JavaScript reduz significativamente alguns cenários tradicionais de XSS.

## O que foi possível demonstrar

Foi possível demonstrar que conteúdo HTML fornecido pelo autor pode permanecer na página pública do currículo.

Entre os elementos aceitos durante os testes está o `<iframe>`, permitindo que uma página externa seja incorporada à página do currículo.

Isso pode representar um risco de segurança dependendo das políticas aplicadas pela plataforma e do conteúdo que pode ser incorporado.

Por exemplo, um visitante pode visualizar conteúdo externo dentro do contexto visual de uma página legítima do Currículo Lattes.

## O que NÃO foi demonstrado

Durante a pesquisa não foi demonstrado:

* execução de JavaScript;
* roubo de cookies;
* acesso a sessões de usuários;
* acesso a dados de outros usuários;
* execução de código no servidor;
* comprometimento da conta de terceiros;
* alteração de currículos de outros usuários.

Todos os testes foram realizados de maneira controlada.

## Evidência técnica

A estrutura observada após o processamento continha um elemento equivalente a:

```html
<iframe src="https://example.com"></iframe>
```

A presença do elemento no HTML final demonstra que o mecanismo de processamento do conteúdo não remove completamente esse tipo de elemento.

Ao mesmo tempo, os testes com JavaScript demonstraram que existem mecanismos de filtragem ou outras proteções impedindo sua execução nos casos testados.

## Divulgação responsável

A vulnerabilidade foi comunicada ao **suporte responsável pelo Currículo Lattes/CNPq**, juntamente com a documentação, evidências e informações necessárias para reproduzir o comportamento.

A comunicação foi realizada antes da publicação desta documentação.

O objetivo deste repositório é registrar tecnicamente a descoberta e contribuir para a identificação e correção do problema.

## Recomendações

A plataforma pode considerar:

1. Tratar o conteúdo do resumo como texto quando HTML não for necessário.
2. Utilizar uma allowlist explícita para as tags permitidas.
3. Validar individualmente os atributos HTML.
4. Restringir ou remover elementos capazes de incorporar conteúdo externo, como `<iframe>`.
5. Restringir as origens externas permitidas caso a incorporação seja necessária.
6. Manter uma política CSP adequada.
7. Criar testes automatizados para garantir que elementos não autorizados permaneçam bloqueados.

## Status

**Descoberta:** confirmada
**Tipo:** Injeção de HTML / sanitização insuficiente
**JavaScript:** não executado nos testes realizados
**Iframe:** demonstrado
**Dados de terceiros:** não acessados
**Teste em terceiros:** não realizado
**Reporte ao suporte:** realizado
**Divulgação:** documentação técnica

## Conclusão

A descoberta ocorreu de maneira acidental durante uma tentativa de personalizar o próprio currículo.

O teste inicial com uma simples tag `<h1>` revelou que o campo de resumo interpreta determinados elementos HTML. Isso levou à investigação da possibilidade de XSS.

Embora os testes realizados não tenham conseguido executar JavaScript — incluindo `<script>` e outras tentativas de execução — foi possível demonstrar a persistência e renderização de um `<iframe>` apontando para conteúdo externo.

A vulnerabilidade já foi comunicada ao suporte do CNPq com a documentação disponível, e esta documentação registra apenas os resultados obtidos durante os testes controlados.
