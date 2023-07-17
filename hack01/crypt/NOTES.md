# Credentials:

Diego Fleury Corrêa de Moraes, 11800584
Vitor Favrin Carrera Miguel, 11800646
Leonardo Vinicius de Almeida, 10392230
Rafael Corona, 4769989
Mathias Fernandes Duarte Coelho, 107343452

## Explicação da descoberta da vulnerabilidade:

Primeiramente temos de contornar o problema de autenticação.
A forma encontrada fora através da exploração do fluxo da lógica
do código binário pelo programa gdb. Os comandos de disassemble 
foram úteis na visualização da função main, e nos levaram a perceber a chamada
à função authorize (outros comandos usados: ni, si, set disassembly-favor intel,
record, fin, print).

Logo após, inspecionamos a reconstrução do código de máquina (dos 
binários docrypt e libauth.so) através do programa objdump para verificar 
quais variáveis, registradoras, parâmetros de pilha estavam sendo 
utilizados pelo código para fazer a autenticação do usuário.
Notou-se que a chamada à authorize faz algum tipo de verificação
que, ao final, retorna simplesmente um valor inteiro como confirmação
da validade do acesso pelo usuário. 

A forma encontrada de burlar o procedimento, seria através da emulação
de um valor verdade igual ao retornado caso tenha sido autenticado 
o usuário. Em outras palavras, caso a lógica seguinte à função 
authorize retorne simplesmente o inteiro que confirma a autenticação
(independente de se foi realizada ou não) então o restante do sistema
pode realizar a decriptação sem maiores problemas.

PS: A registradora EAX fora observada como aquela que guarda o status
da função authorize. 

## Explicação do *bypass*:

Como a biblioteca libauth.so está sendo chamada dinâmicamente, 
e temos como fato que a linkagem dinâmica apenas faz uso do 
nome da função (sem verificação de sua assinatura, ou seja,
quais parâmetros ela recebe) existe a possibilidade da simples
redireção da authorize usada efetivamente por docrypt, sendo 
não mais a versão presente em libauth.so usada, mas agora, uma 
nova versão (que sempre retorna valor verdade de autorização como 
verdadeiro) que possibilita **SEMPRE** a autenticação.

A diretiva run do Makefile automaticamente redireciona para esta
versão de authorize, de modo que o usuário não precisa estar ciente
deste mecanismo, e simplesmente fazer uso da ferramenta.

## Explicação da recuperação da chave de decriptação:

Como a criptografia faz uso de uma chave estática, que é uma 
string armazenada no código de docrypt, utilizamos esta vulnerabilidade
de design para observar a secção da ABI reservada para o armazenamento 
de dados estáticos (através da análise da saída de objdump -x .rodata docrypt),
em que encontramos uma série de strings de prompt, e outras, sendo uma 
delas a chave ("easy").

O comando run do programa make automaticamente substitui a chave de
decodificação por esta encontrada, porém outra pode ser especificada
caso venha a ser necessário.

## Recomendações técnicas para melhorias dos aspectos de robustez e segurança da aplicação.

* O uso de uma chave criptográfica como sendo uma string estática é 
um risco grande para a segurança, dado que é possivel a simples
inspeção da secção correspondente de dados. Caso fosse uma variável
global, ou inicialmente com valor não inicializado não teríamos 
uma mitigação do problema, dado que apenas a secção de código 
em que deve-se investigar muda. 

* A linkagem dinâmica, sem nenhuma verificação da origem das funções
em que temos de acessar em runtime, é outro risco, dada a possibilidade
de redirecionamento de funções, visto que temos conhecimento de seu nome (pois o 
binário não está *stripped* dos símbolos), permitindo assim a inserção 
de software malicioso.
