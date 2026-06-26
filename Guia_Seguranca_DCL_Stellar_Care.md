# Guia de Segurança e Controle de Acesso (DCL) do Banco de Dados: Stellar Care

Este documento estabelece as diretrizes de segurança e governança de acesso para o **Stellar Care**, protegendo o aplicativo de monitoramento clínico e garantindo que o registro de evolução de enfermagem em ambientes hospitalares ocorra de forma segura, auditável e confiável. O princípio fundamental adotado é o do Menor Privilégio.

## 1. Definição de Perfis (Roles)
A primeira etapa para estruturar o controle do hospital é criar os papéis específicos para os profissionais de saúde que utilizarão o sistema.

```sql
-- Criação dos perfis de acesso baseados nos cargos do hospital
-- Creation of access roles based on hospital job positions
CREATE ROLE tecnico;
CREATE ROLE enfermeiro;
CREATE ROLE medico;
CREATE ROLE admin_sistema;
```

## 2. Concessão de Permissões Iniciais (GRANT)
Técnicos e enfermeiros precisam de acesso rápido e confiável para acompanhar pacientes e registrar aferições, como sinais vitais. O acesso inicial fornece as permissões básicas de leitura e criação de registros.

```sql
-- Concedendo acesso de leitura ao cadastro de pacientes
-- Granting read access to the patient registry
GRANT SELECT ON pacientes TO tecnico, enfermeiro;

-- Concedendo acesso para visualizar e inserir novas evoluções no prontuário
-- Granting access to view and insert new evolutions into the medical record
GRANT SELECT, INSERT ON prontuarios TO tecnico;
```

## 3. Garantia de Imutabilidade do Prontuário (REVOKE)
Para garantir a absoluta integridade do histórico do paciente, o sistema deve operar como um registro imutável de eventos clínicos. Uma vez que o técnico registra os sinais vitais ou procedimentos, essa informação não pode ser sobrescrita ou apagada. 

```sql
-- Removendo permissões de alteração e exclusão do técnico para garantir a imutabilidade do prontuário
-- Removing update and delete permissions from the technician to ensure medical record immutability
REVOKE UPDATE, DELETE ON prontuarios FROM tecnico;
```

## 4. Regras de Acesso Elevado e Contenção de Riscos
Profissionais com maior nível de responsabilidade, como enfermeiros-chefes ou médicos, podem ter permissões de atualização para anexar laudos ou retificar documentações específicas, mas a exclusão de dados clínicos é bloqueada em todos os níveis assistenciais para fins de compliance.

```sql
-- Concedendo permissões de inserção e atualização para médicos e enfermeiros
-- Granting insert and update permissions for doctors and nurses
GRANT SELECT, INSERT, UPDATE ON prontuarios TO enfermeiro;
GRANT SELECT, INSERT, UPDATE ON prontuarios TO medico;

-- Revogando a exclusão de dados clínicos de qualquer profissional de saúde
-- Revoking clinical data deletion from any healthcare professional
REVOKE DELETE ON prontuarios FROM enfermeiro, medico;

-- Apenas a administração do sistema pode deletar registros (sob auditoria rigorosa)
-- Only system administration can delete records (under strict auditing)
GRANT DELETE ON prontuarios TO admin_sistema;
```

### Boas Práticas Adicionais
* **Auditoria Contínua:** Certifique-se de que cada instrução de `INSERT` ou `UPDATE` grave automaticamente o ID do usuário e a *timestamp* exata da ação.
* **Offline-first:** Como o Stellar Care é voltado para uso offline-first, a sincronização do cache local com o banco de dados principal deve respeitar rigorosamente essas regras de DCL no momento do envio das transações (sync) para o servidor.
