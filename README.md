# GenAI - Automação de serviços AWS

# Comandos

### Ollama
```bash
docker compose exec ollama ollama pull llama3.2
```

### Conferir que está no FLOCI

```bash
aws sts get-caller-identity
```

### Criar bucket
```bash
aws s3 mb s3://teste-floci
```

### Testar modelo

```bash
aws bedrock-runtime converse \
  --model-id anthropic.claude-3-haiku-20240307-v1:0 \
  --messages '[
    {
      "role": "user",
      "content": [
        {
          "text": "Em uma frase: o que é IaaS?"
        }
      ]
    }
  ]'
```

# confirme que está no Floci
aws sts get-caller-identity # conta 000000000000

# dois buckets S3
aws s3 mb s3://relatorios
aws s3 mb s3://imagens-drone
echo "Inspeção talhão 12: sem pragas" | aws s3 cp - s3://relatorios/inspecao-001.txt

# Uma tabela DynamoDB

aws dynamodb create-table --table-name sensores \
--attribute-definitions AttributeName=id,AttributeType=S \
--key-schema AttributeName=id,KeyType=HASH \
--billing-mode PAY_PER_REQUEST

# um usuário IAM
aws iam create-user --user-name tecnico-campo

# conferir
aws s3 ls
aws dynamodb list-tables
aws iam list-users

# Utilização do MCP da AWS

Com base nas informações oficiais da AWS (via ferramenta de busca do MCP da AWS), aqui está a diferença entre Security Group e Network ACL (NACL):
Diferença principal

Característica	Security Group (SG)	Network ACL (NACL)
Nível de operação	Instância (nível de instância)	Subnet (nível de subnet)
Escopo	Aplica-se a todas as instâncias associadas ao security group	Aplica-se a todas as instâncias nas subnets associadas
Tipo de regra	Apenas allow	Allow e deny
Avaliação	Avalia todas as regras antes de decidir se o tráfego é liberado	Avalia as regras em ordem crescente até encontrar uma correspondência
Retorno (return traffic)	Automaticamente permitido (comportamento stateful)	Deve ser permitido explicitamente (comportamento stateless)


Pontos-chave (fonte: AWS / VPC User Guide)
- Security groups são firewalls virtuais na camada de instância: controlam o tráfego de entrada e saída, rastreando o estado das conexões (por isso o tráfego de retorno é liberado automaticamente).
- Network ACLs operam na camada de subnet e controlam o tráfego de entrada/saída da subnet. São stateless — cada pacote precisa ser explicitamente liberado nas direções de entrada e saída.
Resumo rápido
- Use Security Groups para controle granular por instância (ex.: SSH só da sua IP).
- Use NACLs como camada adicional de defesa em nível de subnet (ex.: bloquear toda uma faixa de IPs em uma subnet pública).
Observações importantes
- As NACLs não filtram tráfego entre instâncias na mesma subnet (não são recomendadas para microsegmentação).
- Por padrão: o security group permite todo o outbound e nega tudo inbound; a NACL padrão permite todo o tráfego.
- Mudanças no security group são aplicadas imediatamente; na NACL, o processo também é rápido, mas a avaliação é diferente (ordem crescente vs. todas as regras).
Fonte: AWS — Compare security groups and network ACLs e AWS VPC FAQs.

# Criando bucket + 2 usuários IAM com políticas distintas

1. Criar bucket
aws s3 mb s3://relatorios

2. Criar o primeiro usuário - somente leitura
aws iam create-user --user-name leitor-s3
aws iam put-user policy \
--user-name leitor-s3 \
--policy-name politica-s3-read-only \
--policy document file://policy-read-only.json

policy-read-only.json:
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket",
        "s3:GetObject"
      ],
      "Resource": [
        "arn:aws:s3:::relatorios",
        "arn:aws:s3:::relatorios/*"
      ]
    }
  ]
}

# ARN Bucket vs ARN objeto

- s3:ListBucket -> roda sobre ARN do bucket (arn:aws:s3:::relatorios)
- s3:GetObject, s3:PutObject, -> rodam sobre o ARN do objeto (arn:aws:s3:::relatorios/*)

# Etapa 3 - Infraestrutura como código gerado por IA.

