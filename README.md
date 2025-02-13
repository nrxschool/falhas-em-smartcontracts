# Vulnerabilidades em Contratos Inteligentes

- MÓDULO 1
    - AULA 1
        
        [Underflow/Overflow](https://hackernoon.com/hack-solidity-integer-overflow-and-underflow)
        
    - AULA 3
        
        [Hack da DAO](https://www.coindesk.com/consensus-magazine/2023/05/09/coindesk-turns-10-how-the-dao-hack-changed-ethereum-and-crypto/)
        
        [Reentrância](https://hackernoon.com/hack-solidity-reentrancy-attack) 
        
        [Reentrância](https://solidity-by-example.org/hacks/re-entrancy/) 
        
    - AULA 4
        
        [Como Mitigar Vulnerabilidades de Controle de Acesso](https://medium.com/rektify-ai/how-to-mitigate-access-control-vulnerability-6df74c82af98) 
        
        [Vulnerabilidades de Controle de Acesso em Contratos Inteligentes Solidity](https://www.immunebytes.com/blog/access-control-vulnerabilities-in-solidity-smart-contracts/)
        
        [Vulnerabilidades de Controle de Acesso](https://medium.com/ginger-security/access-control-vulnerabilities-in-solidity-smart-contracts-5e0871a00d77) 
        
    - AULA 5
        
        [Front Running](https://solidity-by-example.org/hacks/front-running/)
        
        [Simulação de Ataque Front-running com Hardhat](https://github.com/pedrosgmagalhaes/frontrunning_attack)
        
        [Como Resolver a Vulnerabilidade de Front-running em Contratos Inteligentes](https://hackernoon.com/how-to-solve-the-frontrunning-vulnerability-in-smart-contracts)

# Simulação de Ataque Front-running com Hardhat

Este repositório contém uma demonstração simplificada de um ataque de front-running na rede Ethereum. Estamos usando o ambiente de desenvolvimento Ethereum [Hardhat](https://hardhat.org/) para esta simulação.

## Visão Geral

No Ethereum, front-running é um tipo de ataque em que uma entidade mal-intencionada tenta se beneficiar ao visualizar os detalhes de uma transação antes que ela seja confirmada. O invasor pode então emitir sua própria transação com um preço de gás mais alto, tornando mais provável que os mineradores incluam a transação do atacante no próximo bloco, sendo processada antes da transação da vítima.

Nesta simulação, consideramos um cenário onde um usuário mal-intencionado antecipa a transação de um usuário comum para modificar um estado compartilhado em um contrato, impedindo a transação original de ser processada.

## Contrato

O contrato `FrontRunningDemo` é bastante simples:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract FrontRunningDemo {
    struct Transaction {
        address user;
        uint256 amount;
    }

    Transaction[] public transactions;

    uint256 public state = 0;

    function submitTransaction(uint256 amount) external {
        require(amount > state, "O valor deve ser maior que o estado atual");

        transactions.push(Transaction(msg.sender, amount));
        state = amount;
    }

    function getTransactions() external view returns (Transaction[] memory) {
        return transactions;
    }
}
```

O contrato mantém um estado (um número) e permite que os usuários enviem uma transação que altera esse estado. No entanto, qualquer transação com um valor menor ou igual ao estado atual será rejeitada.

## Teste
Nosso teste simula o cenário do ataque:

```javascript
it("deve executar um ataque de front-running", async function () {
    // Usuário A prepara uma transação
    const ownerAmount = ethers.utils.parseUnits("20", "ether");

    // Atacante B observa a transação do Usuário A e prepara sua própria transação
    const attackerAmount = ethers.utils.parseUnits("100", "ether");

    // Atacante B antecipa a transação do Usuário A
    await frontRunningDemo.connect(attacker).submitTransaction(attackerAmount);

    // Usuário A envia sua transação, que deve falhar
    try {
        await frontRunningDemo.connect(owner).submitTransaction(ownerAmount);
        throw new Error("A transação do proprietário não falhou como esperado");
    } catch (error) {
        if (error.message.includes("A transação do proprietário não falhou como esperado")) {
            throw error;
        } else {
            console.log("A transação do proprietário falhou como esperado");
        }
    }

    // Verificar a ordem das transações
    const transactions = await frontRunningDemo.getTransactions();
    expect(transactions.length).to.equal(1);
    expect(transactions[0].user).to.equal(attackerAddress); // Apenas a transação do atacante deve ter sido bem-sucedida
    expect(transactions[0].amount.toString()).to.equal(attackerAmount.toString());
});
```

Esse teste simula um ataque de front-running no ambiente local do Hardhat. O invasor observa a transação de um usuário comum e a antecipa enviando sua própria transação primeiro. A transação do usuário comum falha, pois não atende aos requisitos do contrato.
