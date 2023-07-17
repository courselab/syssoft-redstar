## Membros:

Guilherme Ramos Costa Paixão - 11796079

## Descoberta da Vulnerabilidade:

Inicialmente, foi necessário enfrentar o problema de autenticação. A solução encontrada consistiu em explorar o fluxo da lógica presente no código binário usando o programa gdb. Com o auxílio de comandos como "disassemble", conseguimos visualizar a função principal (main) e identificamos a chamada para a função "authorize". Outros comandos utilizados incluíram "ni", "si", "set disassembly-favor intel", "record", "fin" e "print".

Em seguida, examinamos a reconstrução do código de máquina dos binários "docrypt" e "libauth.so" usando o programa "objdump". O objetivo era verificar quais variáveis, registradores e parâmetros de pilha estavam sendo utilizados pelo código para realizar a autenticação do usuário. Observou-se que a função "authorize" realizava algum tipo de verificação e, ao final, simplesmente retornava um valor inteiro como confirmação da validade do acesso do usuário.

A maneira de contornar esse procedimento foi emular um valor verdadeiro que fosse igual ao valor retornado quando o usuário era autenticado. Em outras palavras, independentemente de o usuário ter sido autenticado ou não, caso a lógica subsequente à função "authorize" retornasse simplesmente o inteiro de confirmação de autenticação, o restante do sistema poderia realizar a decriptação sem problemas adicionais.

Observação: A registradora "EAX" foi identificada como aquela que armazena o status da função "authorize".

## Bypass da Vulnerabilidade:

Como a biblioteca "libauth.so" é chamada dinamicamente e a linkagem dinâmica usa apenas o nome da função (sem verificar seus parâmetros), existe a possibilidade de simplesmente redirecionar a função "authorize" efetivamente utilizada pelo "docrypt". Dessa forma, a versão presente em "libauth.so" não é mais usada; em vez disso, uma nova versão é utilizada (que sempre retorna um valor verdadeiro de autorização), permitindo que a autenticação ocorra SEMPRE.

A diretiva "run" no Makefile redireciona automaticamente para essa nova versão da função "authorize", tornando o usuário inconsciente desse mecanismo e possibilitando o uso simples da ferramenta.
Recuperação da Chave de Decriptação:

Como a criptografia utiliza uma chave estática, que é uma string armazenada no código de "docrypt", aproveitamos essa vulnerabilidade de design para observar a seção da ABI (Application Binary Interface) reservada para o armazenamento de dados estáticos. Através da análise da saída do comando "objdump -x .rodata docrypt", identificamos uma série de strings, incluindo a chave ("easy").

O comando "run" do programa "make" automaticamente substitui a chave de decodificação pela encontrada no binário, mas outra chave pode ser especificada, se necessário.

## Recomendações para Melhorias de Robustez e Segurança da Aplicação:

- Utilizar uma chave criptográfica como uma string estática representa um grande risco à segurança, pois a seção de dados correspondente pode ser facilmente inspecionada. Caso a chave fosse uma variável global ou inicializada dinamicamente, o problema não seria mitigado, pois apenas a seção de código a ser investigada seria diferente.

- A linkagem dinâmica, sem qualquer verificação da origem das funções acessadas em tempo de execução, é outro risco significativo, pois possibilita o redirecionamento de funções conhecidas por seus nomes (já que o binário não foi "stripped" dos símbolos). Isso pode permitir a inserção de software malicioso na aplicação.
