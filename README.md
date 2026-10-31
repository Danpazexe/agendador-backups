# agendador-backups

**Agendador semanal do backup do DietSpace. Nenhum dado fica neste repositório.**

[![Backup DietSpace](https://github.com/Danpazexe/agendador-backups/actions/workflows/dietspace.yml/badge.svg)](https://github.com/Danpazexe/agendador-backups/actions/workflows/dietspace.yml)
![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-agendado-2088FF?logo=githubactions&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-18-4169E1?logo=postgresql&logoColor=white)
![Criptografia](https://img.shields.io/badge/AES--256-OpenSSL-721412?logo=openssl&logoColor=white)

O dump do banco é criptografado (AES-256) ainda no runner e enviado para um repositório **privado**. Aqui fica só o workflow.

## Como funciona

O workflow [`dietspace.yml`](.github/workflows/dietspace.yml) roda todo **domingo às 15h UTC** (12h em Brasília) e também pode ser disparado à mão em **Actions → Backup DietSpace → Run workflow**.

1. Mascara o host do banco nos logs.
2. Instala o cliente do PostgreSQL 18.
3. Gera o dump (`pg_dump -Fc`) e o criptografa com `openssl enc -aes-256-cbc -pbkdf2` (200 mil iterações).
4. Confere o arquivo: descriptografa e lê com `pg_restore` antes de enviar.
5. Envia para o repositório privado por deploy key, mantendo os **12 backups** mais recentes.
6. Atualiza [`ultima-execucao.txt`](ultima-execucao.txt). Esse commit mantém o agendamento ativo, já que o GitHub desliga workflows agendados de repositórios sem atividade por 60 dias.

## Secrets

| Secret | Para quê |
|--------|----------|
| `NEON_DATABASE_URL` | Conexão com o banco de origem |
| `BACKUP_PASSPHRASE` | Senha da criptografia (guarde uma cópia fora do GitHub) |
| `DIETSPACE_BACKUPS_DEPLOY_KEY` | Deploy key com escrita no repositório de destino |

## Restaurar

```sh
openssl enc -d -aes-256-cbc -pbkdf2 -iter 200000 -in dietspace-AAAA-MM-DD-HHMM.dump.enc -out dietspace.dump
pg_restore --no-owner --no-privileges -d "$DATABASE_URL" dietspace.dump
```

## Licença

Uso interno da [PazConcept](https://www.pazconcept.com.br). Todos os direitos reservados.
