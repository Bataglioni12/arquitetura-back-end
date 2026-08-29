# Comparação entre contrato, contrato gerado e execução

O contrato explícito (OpenAPI) descreve formalmente os endpoints, parâmetros, schemas e códigos de resposta esperados da API.

O contrato gerado pela FastAPI, acessível em `/docs` e `/openapi.json`, representa a documentação produzida a partir da implementação e permite visualizar e testar as operações disponíveis.

A execução prática, realizada com Bruno, confirmou os comportamentos definidos no contrato. O POST retornou HTTP 202 com protocolo e cabeçalho Location. O GET recuperou corretamente o recurso criado com HTTP 200.

Como falha deliberada, foi enviado um POST sem o campo `cpf`. O sistema respondeu com HTTP 422 e identificou o campo ausente (`body.cpf`). Essa validação demonstra a conformidade entre contrato e implementação, não sendo um defeito pendente.