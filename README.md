CONTEUDO ORIGINAL OBTIDO DE:

https://marcelog.github.io/articles/asterisk_json_curl_dialplan.html

https://github.com/drivefast/asterisk-res_json

https://gist.github.com/swchris

Com anotações e adequações especificas para projetos da intervia 

# Curl-e-Json-do-Asterisk-Dialplan
Como usar Curl e Json do Asterisk Dialplan para controlar o fluxo de chamadas

É muito comum que queiramos usar serviços externos do nosso Asterisk Dialplan , e muitas vezes esses serviços externos são acessíveis via HTTP (como uma REST HTTP API ).

Neste artigo, veremos como podemos usar o cURL para consultar um serviço HTTP externo, ler uma resposta no formato JSON e agir sobre os valores retornados para controlar o fluxo de chamadas em nosso dialplan.

Usando cURL para consultar uma API HTTP do Asterisk Dialplan
O Asterisk tem a função dialplan CURL que pode ser usada para emitir solicitações HTTP do dialplan.

exten => _X.,1,Set(CURL_RESULT=${CURL(http://domain.com/test.txt)})
view rawextensions.conf hosted with ❤ by GitHub
Uma vez feita a solicitação, podemos acessar o resultado (o corpo da solicitação) na variável CURL_RESULT , usando o dialplan Set Application para definir o valor da variável .

Usando HTTPS para consultar APIs REST do dialplan
Caso você precise acessar um endpoint HTTPS com um certificado autoassinado ou outros problemas de SSL que podem causar falha na solicitação HTTPS, você pode usar a função dialplan CURLOPT para desabilitar o ssl_verifypeer no cURL antes de fazer a solicitação:

same => n,Set(CURLOPT(ssl_verifypeer)=0)
view rawextensions.conf hosted with ❤ by GitHub
Usando o resultado da solicitação HTTP para bifurcar no Asterisk Dialplan
Utilizando o aplicativo GotoIf do Asterisk Dialplan é possível tomar medidas dependendo do valor retornado pela requisição HTTP:

exten => _X.,1,Set(CURL_RESULT=${CURL(http://domain.com/test.txt)})
same => n,GotoIf($["${CURL_RESULT}" = "1"]?result1:result2)
same => n(result1),Verbose(Result 1)
same => n,Hangup
same => n(result2),Verbose(Result other)
same => n,Hangup
view rawextensions.conf hosted with ❤ by GitHub
O código acima emitirá uma requisição para o arquivo test.txt, e dependendo do valor exato, bifurcará a execução para o rótulo result1 ou result2 , o que é bem útil! Lembre-se de que estamos verificando o valor exato (conteúdo) da requisição, que neste caso é o conteúdo do arquivo test.txt .

Usando res_json no Asterisk Dialplan para processar uma string JSON
É bem comum que APIs retornem um payload JSON, e podemos usar res_json em nosso dialplan para analisar o conteúdo que obtemos como resultado de uma solicitação HTTP e agir de acordo. Isso pode ser obtido compilando e instalando res_json de acordo com as instruções ( https://github.com/drivefast/asterisk-res_json ) e, em seguida, a partir do dialplan:

exten => _X.,1,Set(CURL_RESULT=${CURL(http://domain.com/test.json)})
same => n,Set(result=${JSONELEMENT(CURL_RESULT,result/subfield)})
same => n,GotoIf($["${result}" = "1"]?result1:result2)
same => n(result1),Verbose(Result 1)
view rawextensions.conf hosted with ❤ by GitHub
Podemos testar isso tendo o seguinte conteúdo para test.json :

{
  "result": {
    "subfield": 1
  }
}
view rawtest.json hosted with ❤ by GitHub
Conclusão
Sem muito esforço, agora podemos chamar APIs HTTP que retornam texto simples ou strings JSON e processá-las em nossos dialplans Asterisk para tomar decisões. Aproveite!
