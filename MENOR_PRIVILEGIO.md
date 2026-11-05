# MENOR_PRIVILEGIO

## Contexto

Reinicialização do Floci com `FLOCI_SERVICES_IAM_ENFORCEMENT_ENABLED=true`.

## Política de IAM aplicada

Usuário criado: `leitor-relatorios`
Política: `LeituraBucketRelatorios`

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowListBucketRelatorios",
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::relatorios"
    },
    {
      "Sid": "AllowReadObjectsRelatorios",
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::relatorios/*"
    }
  ]
}
```

## Usuário criado

- **UserName:** `leitor-relatorios`
- **UserId:** `AIDAYA5WJVTEN9EHR5CA`
- **Arn:** `arn:aws:iam::000000000000:user/leitor-relatorios`

## Chave de acesso

- **AccessKeyId:** `AKIA97YDMR9LLVRX58IY`
- **Status:** Active

## Testes realizados

### 1. Get Object — bucket `relatorios`, chave `inspecao-001.txt`

**Comando:**
```bash
AWS_ACCESS_KEY_ID=AKIA97YDMR9LLVRX58IY AWS_SECRET_ACCESS_KEY=WTq8fTSnPwRaKq8+b8YujdBXMqefvu/wU/33TEpE \
  aws s3api get-object --bucket relatorios --key inspecao-001.txt /tmp/opencode/get-out.txt
```

**Resultado:** **PASSOU** (exit 0)
- Objeto recuperado com sucesso (ContentLength: 34 bytes)

{
    "AcceptRanges": "bytes",
    "LastModified": "2026-11-05T13:14:23+00:00",
    "ContentLength": 34,
    "ETag": "\"f120eb008eed1538dfe232923580c035\"",
    "ChecksumCRC64NVME": "sx/2Al0V7Wk=",
    "ChecksumType": "FULL_OBJECT",
    "ContentType": "text/plain",
    "Metadata": {},
    "StorageClass": "STANDARD"
}


### 2. Put Object — bucket `relatorios`, chave `inspecao-001-bloqueado.txt`

**Comando:**
```bash
AWS_ACCESS_KEY_ID=AKIA97YDMR9LLVRX58IY AWS_SECRET_ACCESS_KEY=WTq8fTSnPwRaKq8+b8YujdBXMqefvu/wU/33TEpE \
  aws s3 cp /tmp/opencode/inspecao-001.txt s3://relatorios/inspecao-001-bloqueado.txt
```

**Resultado:**  **FALHOU** (exit 1)
- Erro: `An error occurred (AccessDenied) when calling the PutObject operation: User is not authorized to perform: s3:PutObject`

## Conclusão

- Usuário `leitor-relatorios` **pode ler** (`s3:GetObject`) o bucket `relatorios`.
-  Usuário `leitor-relatorios` **não pode escrever** (`s3:PutObject`) no bucket `relatorios`.

**POLÍTICA DE MENOR PRIVILEGIO VALIDADA:** a política de leitura (somente `GetObject`/`ListBucket`) está correta e funcionando com o enforcement de IAM ativo.
