# cursodezkp
Cruso de ZKP - Circom

## TAU

Baixar arquivo TAU da Hermez [arquivo](https://storage.googleapis.com/zkevm/ptau/powersOfTau28_hez_final_15.ptau) [referencia](https://github.com/iden3/snarkjs?tab=readme-ov-file) que suporta 32k de constraints.

## Instalação das ferramentas

### Circom

https://docs.circom.io/getting-started/installation/

### Snarkjs

https://github.com/iden3/snarkjs

### Referencia tecnica

https://docs.circom.io/getting-started/installation/

### Comandos para multiplicador

#### Passo 1: Compilação do circuito
```shell
cd exemplo-multiplicador

# Compila o circuito Circom gerando:
# - multiplicador.r1cs: representação R1CS (Rank-1 Constraint System) do circuito
# - multiplicador.wasm: arquivo WebAssembly para gerar testemunhas
# - multiplicador.sym: arquivo de símbolos para debugging
circom multiplicador.circom --r1cs --wasm --sym
```

#### Passo 2: Setup da cerimônia de confiança (Trusted Setup)
```shell
# Gera a chave inicial usando o arquivo TAU pré-computado
# Cria multiplicador_00.zkey: chave de prova inicial
snarkjs groth16 setup multiplicador.r1cs ../powersOfTau28_hez_final_15.ptau multiplicador_00.zkey

# Adiciona uma contribuição à cerimônia de confiança
# Gera multiplicador_01.zkey: chave de prova com contribuição aleatória
snarkjs zkey contribute multiplicador_00.zkey multiplicador_01.zkey --name="1a contribuicao"

# Extrai a chave de verificação pública
# Gera chave-verificacao.json: chave pública para verificar provas
snarkjs zkey export verificationkey multiplicador_01.zkey chave-verificacao.json
```

#### Passo 3: Geração e verificação de provas
```shell
# Gera a testemunha (witness) com os valores de entrada
# Cria testemunha.wtns: valores secretos que satisfazem o circuito
node multiplicador_js/generate_witness.js multiplicador_js/multiplicador.wasm valores-entrada.json testemunha.wtns

# Gera a prova ZK usando Groth16
# Cria prova.json: prova criptográfica
# Cria valores-entrada-publica.json: inputs públicos da prova
snarkjs groth16 prove multiplicador_01.zkey testemunha.wtns prova.json valores-entrada-publica.json

# Verifica se a prova é válida
# Retorna true se a prova está correta
snarkjs groth16 verify chave-verificacao.json valores-entrada-publica.json prova.json 
```

#### Passo 4: Exportação para Solidity
```shell
# Gera contrato Solidity para verificação on-chain
# Cria verificador.sol: contrato inteligente para verificar provas
snarkjs zkey export solidityverifier multiplicador_01.zkey verificador.sol

# Gera dados formatados para chamada do contrato Solidity
# Cria valores-entrada-solidity.txt: parâmetros formatados para o contrato
snarkjs zkey export soliditycalldata valores-entrada-publica.json prova.json > valores-entrada-solidity.txt
```

### Comandos para validador conta

#### Passo 1: Compilação do circuito
```shell
cd exemplo-validaconta

# Compila o circuito Circom gerando:
# - validaconta.r1cs: representação R1CS (Rank-1 Constraint System) do circuito
# - validaconta.wasm: arquivo WebAssembly para gerar testemunhas
# - validaconta.sym: arquivo de símbolos para debugging
circom validaconta.circom --r1cs --wasm --sym
```

#### Passo 2: Setup da cerimônia de confiança (Trusted Setup)
```shell
# Gera a chave inicial usando o arquivo TAU pré-computado
# Cria validaconta_00.zkey: chave de prova inicial
snarkjs groth16 setup validaconta.r1cs ../powersOfTau28_hez_final_15.ptau validaconta_00.zkey

# Adiciona uma contribuição à cerimônia de confiança
# Gera validaconta_01.zkey: chave de prova com contribuição aleatória
snarkjs zkey contribute validaconta_00.zkey validaconta_01.zkey --name="1a contribuicao"

# Extrai a chave de verificação pública
# Gera chave-verificacao.json: chave pública para verificar provas
snarkjs zkey export verificationkey validaconta_01.zkey chave-verificacao.json
```

#### Passo 3: Geração e verificação de provas
```shell
# Gera a testemunha (witness) com os valores de entrada
# Cria testemunha.wtns: valores secretos que satisfazem o circuito
node validaconta_js/generate_witness.js validaconta_js/validaconta.wasm valores-entrada.json testemunha.wtns

# Gera a prova ZK usando Groth16
# Cria prova.json: prova criptográfica
# Cria valores-entrada-publica.json: inputs públicos da prova
snarkjs groth16 prove validaconta_01.zkey testemunha.wtns prova.json valores-entrada-publica.json

# Verifica se a prova é válida
# Retorna true se a prova está correta
snarkjs groth16 verify chave-verificacao.json valores-entrada-publica.json prova.json 
```

#### Passo 4: Exportação para Solidity
```shell
# Gera contrato Solidity para verificação on-chain
# Cria verificador.sol: contrato inteligente para verificar provas
snarkjs zkey export solidityverifier validaconta_01.zkey verificador.sol

# Gera dados formatados para chamada do contrato Solidity
# Cria valores-entrada-solidity.txt: parâmetros formatados para o contrato
snarkjs zkey export soliditycalldata valores-entrada-publica.json prova.json > valores-entrada-solidity.txt
```

## reference 

o curso foi ministrado por [JeffPrestes](https://www.linkedin.com/in/jeffprestes/), esse é um repositório com modificações p/ consulta futura.