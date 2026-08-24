# Observações

**Condição alterada:** Foi alterado o horário da consulta que gerava conflito, passando de 09:15-09:45 para 11:00-11:30.

**O que a saída revelou:** Antes da alteração, o sistema retornava HTTP 409 devido à sobreposição de horários. Após a alteração, o agendamento foi realizado com sucesso, retornando HTTP 201.

**Responsabilidade arquitetural relacionada:** A evidência está associada à camada de serviços (`servicos.py`), responsável por aplicar as regras de negócio e validar conflitos de agenda.
