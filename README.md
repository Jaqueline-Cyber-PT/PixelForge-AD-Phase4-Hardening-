## Fase 4- Hardening, Remediação e Prevenção Ativa

### **Habilidades Demonstradas**

- Hardening de Active Directory
- Gestão de GPO (Group Policy Objects)
- Proteção de Identidade e Memória
- Segurança de Protocolos de Rede
- Auditoria Avançada e Visibilidade

### **1. Introdução: Estratégia de Resiliência**

Após a identificação de vulnerabilidades críticas e o comprometimento total do domínio nas fases anteriores, este projeto marca a transição para a **defesa ativa**. O objetivo central é implementar um plano de remediação baseado no endurecimento (**Hardening**) da infraestrutura do Active Directory. Esta fase foca em converter as lições aprendidas durante o Pentest em controles técnicos robustos, garantindo que o ambiente **PF-DC01** não seja apenas restaurado, mas blindado contra os vetores de ataque explorados anteriormente, como *Password Spraying* e interceptação de rede.

**2. Implementação de Baseline de Segurança (GPO & Identidade)**

A primeira linha de defesa consistiu-se na reestruturação das políticas de autenticação do domínio. Utilizando automação via **PowerShell**, apliquei uma nova **Baseline de Segurança** que elevou o requisito mínimo de senhas para **12 caracteres** com complexidade obrigatória. Além disso, foi implementado o **Account Lockout Threshold**, configurado para bloquear contas após **5 tentativas falhas** em um intervalo de 30 minutos. Esta medida neutraliza tecnicamente ataques de força bruta e *Password Spraying*, forçando a interrupção de ferramentas automatizadas assim que o limite de erros é atingido.

<img width="1141" height="454" alt="image" src="https://github.com/user-attachments/assets/f3f0c93a-941b-42e3-af29-d42761257a63" />

### **3. Hardening Sistêmico e Redução de Superfície de Ataque**

Nesta etapa, foi realizada a consolidação de controles voltados à redução da superfície de ataque, com foco na eliminação de vetores comuns de exploração em ambientes Windows.

Foram desativados protocolos legados de resolução de nomes, como **LLMNR e NetBIOS**, frequentemente explorados em ataques de *Name Poisoning* e *Man-in-the-Middle (MITM)*. A remoção desses serviços reduz significativamente a possibilidade de interceptação de credenciais em redes internas.

Adicionalmente, foi aplicada a proteção do processo **LSASS** por meio da diretiva **LSA Protection (RunAsPPL)**, que impede a leitura direta da memória por ferramentas de *credential dumping*,  elevando o nível de proteção das credenciais, desativando o armazenamento de senhas em texto claro na memória **(WDigest)** e ****mitigando o uso de ferramentas como Mimikatz. Essa abordagem garante uma camada adicional de defesa em profundidade, dificultando técnicas comuns de pós exploração utilizadas por atacantes.

<img width="1465" height="160" alt="image" src="https://github.com/user-attachments/assets/5310e9cc-e191-4c40-841e-c3412e6b650f" />

### **4. Governança e Visibilidade (Auditoria Avançada)**

Com o objetivo de aumentar a capacidade de detecção e resposta a incidentes, foram implementadas políticas avançadas de auditoria e monitoramento.

Foi habilitado o **Script Block Logging**, permitindo registrar a execução de comandos PowerShell, inclusive aqueles ofuscados, o que é essencial para análise forense garantindo que mesmo os scripts executados em memória sejam inspecionados pela **AMSI (Antimalware Scan Interface)** e auxiliando na identificação de atividades maliciosas.

Além disso, foram configuradas políticas de auditoria para eventos críticos, como: Criação de processos e Eventos de logon. Esses registros aumentam significativamente a visibilidade do ambiente, permitindo a identificação de comportamentos anômalos e rastreabilidade de ações executadas no sistema.

<img width="1354" height="668" alt="image" src="https://github.com/user-attachments/assets/b1843c07-eca3-4cab-85f4-80ebb65b3e48" />

### **5. Segurança de Protocolos de Rede e Diretório**

Para mitigar ataques de interceptação e reencaminhamento de autenticação, foram aplicados controles de integridade nos principais protocolos de rede.

A obrigatoriedade da Assinatura Digital de SMB **(SMB Signing)** foi configurada via registro (`RequireSecuritySignature`), garantindo que toda comunicação SMB seja validada quanto à sua integridade e autenticidade. Essa medida neutraliza ataques de **SMB Relay**, impedindo que um invasor manipule sessões autenticadas.

Complementarmente, foi implementada a exigência de **LDAP Signing**, protegendo o controlador de domínio contra ataques de **NTLM Relay** direcionados ao serviço de diretório. Essas configurações fortalecem a segurança da comunicação interna e garantem maior confiabilidade nos processos de autenticação.

<img width="1474" height="105" alt="image" src="https://github.com/user-attachments/assets/3e8df724-0d8a-4cde-81cb-d739424483c8" />

### **6. Validação de Controles: Perspectiva do Atacante (Pentest Audit)**

A eficácia das configurações de *hardening* foi validada através de uma perspectiva externa (atacante), utilizando ferramentas de auditoria de rede para confirmar a ausência de vetores de exploração.

- **6.1. Auditoria de Dialetos SMB (Nmap)**
    
    Utilizando o **Nmap** com o script `smb-protocols`, foi verificado que o servidor não oferece mais suporte ao protocolo **SMBv1/CIFS**. O ambiente opera estritamente com dialetos modernos (SMB 2.x e 3.x), garantindo criptografia e integridade na comunicação de arquivos.

  <img width="954" height="334" alt="image" src="https://github.com/user-attachments/assets/fb3568e4-36bf-4dc6-b0bc-1707f0671b60" />

- **6.2. Verificação de Integridade e Relay (NetExec)**
    
    Através da ferramenta **NetExec (nxc)**, foi confirmada a obrigatoriedade de assinatura digital em todas as comunicações SMB. Esta configuração é crítica para impedir ataques de **SMB Relay**, onde um adversário poderia interceptar uma autenticação legítima e reencaminhá-la para obter acesso administrativo.

<img width="1016" height="96" alt="image" src="https://github.com/user-attachments/assets/c149dcfe-c94c-4758-9b55-73aa1e7d5edc" />

### **7. Conclusão Geral: Da Vulnerabilidade à Resiliência**

Este projeto demonstrou o ciclo completo de segurança em um ambiente de Active Directory, desde a exploração inicial de falhas críticas até a implementação de um ecossistema blindado. A transição do ambiente **PF-DC01** de um estado vulnerável para uma infraestrutura endurecida reforça que a segurança não depende de uma única "solução mágica", mas de uma estratégia de **Defesa em Profundidade**.

As remediações aplicadas como: a proteção do processo LSASS, a obrigatoriedade de assinaturas digitais (SMB/LDAP) e a visibilidade avançada via Script Block Logging, transformaram vetores de ataque anteriormente silenciosos em trilhas de auditoria detectáveis. O resultado final é um domínio que não apenas resiste a ataques de *Relay* e *Credential Dumping*, mas que também impõe um custo computacional e operacional proibitivo para qualquer adversário.

## **8. Future Enhancements (Melhorias Futuras)**

Embora a fase atual tenha estabelecido uma base sólida de *Hardening*, a evolução das ameaças exige uma postura de vigilância contínua. Em uma etapa subsequente de maturidade (Fase 5), as seguintes implementações seriam priorizadas:

- **Integração com SIEM (Security Information and Event Management):** Centralização dos logs de auditoria (Event IDs 4688, 4104, 4625) em uma plataforma como **Microsoft Sentinel** ou **ELK Stack** para criação de alertas em tempo real.
- **Implementação de EDR (Endpoint Detection and Response):** Monitoramento comportamental avançado nas estações de trabalho e servidores para detectar anomalias que burlam assinaturas de antivírus tradicionais.
- **Estratégias de Deception (Honeytokens):** Criação de usuários e objetos falsos no Active Directory com privilégios atraentes, servindo como "armadilhas" para detectar intrusos na fase de enumeração.
- **Privileged Access Management (PAM):** Implementação de uma solução para gerenciamento de contas de alto privilégio com senhas rotativas e sessões monitoradas (Just-in-Time Access).
- **Tiered Administration Model:** Segregação total de contas administrativas por níveis de criticidade (Tier 0, 1 e 2), garantindo que credenciais de Domain Admin nunca toquem estações de trabalho de usuários comuns.
