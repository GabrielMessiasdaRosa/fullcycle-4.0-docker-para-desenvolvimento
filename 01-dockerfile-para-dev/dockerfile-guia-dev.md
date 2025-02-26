# Dockerfile para Desenvolvimento: Análise Detalhada

## 1. Estrutura Básica da Imagem

```dockerfile
FROM node:22.8.0-slim
```

- **Objetivo**: Usar uma imagem leve e otimizada.
- **Detalhes**:
  - \`slim\`: Reduz tamanho da imagem (~20% menor que a padrão).
  - Mantém compatibilidade com \`glibc\` para evitar problemas com binários.

---

## 2. Gerenciamento de Dependências

```dockerfile
ARG NODEMON_VERSION=3.1.7
RUN apt update && \\
    apt install -y curl && \\
    npm install -g nodemon@${NODEMON_VERSION}
```

- **Boas Práticas**:
  - Uso de \`ARG\` para parametrizar versões facilita atualizações.
  - \`apt update\` + \`apt install\` em um único \`RUN\` reduz camadas da imagem.
- **Atenção**:
  - Remova o \`curl\` após a instalação se não for necessário:
    ```dockerfile
    RUN apt update && \\
        apt install -y --no-install-recommends curl && \\
        npm install -g nodemon@${NODEMON_VERSION} && \\
        apt purge -y curl && \\
        apt autoremove -y
    ```

---

## 3. Criação de Usuário Não-Root

### Opção Debian:

```dockerfile
# RUN useradd -m -u 1000 xpto
```

- **Vantagens**:
  - Mantém compatibilidade com sistemas que usam \`glibc\`.
  - Ideal para imagens baseadas em Debian/Ubuntu.

### Opção Alpine:

```dockerfile
# RUN adduser -D -u 1000 xpto
```

- **Vantagens**:
  - Mais leve (~5 MB para a imagem base).
  - Recomendado para imagens otimizadas ao extremo.

---

## 4. Configuração do Ambiente

```dockerfile
COPY start.sh /
RUN chmod +x /start.sh
USER node
WORKDIR /home/node/app
EXPOSE 3000
```

- **Detalhes**:
  - \`chmod +x\`: Garante permissão de execução ao script.
  - \`USER node\`: Segurança contra execução como root (usuário padrão da imagem Node.js).
  - \`WORKDIR\`: Isola o código da aplicação em diretório dedicado.

---

## 5. Inicialização do Container

```dockerfile
CMD ["/start.sh"]
# ENTRYPOINT [ "executable" ]
```

- **Diferenças**:  
  | **Instrução** | **Comportamento** | **Sobrescrita** |  
  |---------------|-----------------------------------------------|--------------------------------|  
  | \`CMD\` | Define comando padrão | Pode ser substituído por flags |  
  | \`ENTRYPOINT\` | Define executável principal | Requer \`--entrypoint\` para mudar |

---

## 6. Script de Inicialização (\`start.sh\`)

- **Conteúdo Sugerido**:

```bash
#!/bin/sh
npm install && npm start
```

- **Funções**:
  - Instala dependências em tempo de execução (útil para desenvolvimento).
  - Inicia a aplicação.

---

## 7. Comparação de Imagens Base

| **Imagem**           | **Tamanho** | **Casos de Uso**             | **Cuidados**                     |
| -------------------- | ----------- | ---------------------------- | -------------------------------- |
| \`node:22.8.0-slim\` | ~180 MB     | Apps com dependências comuns | Verificar compatibilidade libc   |
| \`node:22-alpine\`   | ~120 MB     | Microsserviços leves         | Problemas com binários \`glibc\` |
| \`node:22-bullseye\` | ~340 MB     | Debugging complexo           | Tamanho elevado                  |

---

## 8. Recomendações de Segurança

1. **Não Use Root**: Sempre execute como usuário não-privilegiado (\`USER node\`).
2. **Verifique Permissões**: Scripts e diretórios devem ter permissões mínimas necessárias.
3. **Atualize Imagens**: Use \`docker scan\` para verificar vulnerabilidades:
   ```bash
   docker scan node:22.8.0-slim
   ```

---

## 9. Dockerfile Otimizado (Exemplo Final)

```dockerfile
FROM node:22.8.0-slim

ARG NODEMON_VERSION=3.1.7

RUN apt update && \
    apt install -y --no-install-recommends curl && \
    npm install -g nodemon@${NODEMON_VERSION} && \
    apt purge -y curl && \
    apt autoremove -y && \
    rm -rf /var/lib/apt/lists/*

COPY start.sh /
RUN chmod +x /start.sh

USER node
WORKDIR /home/node/app
EXPOSE 3000

CMD ["/start.sh"]
```