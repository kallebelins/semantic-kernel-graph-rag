## UC01 — GraphCase: Upload/ZIP/GDrive → Normalize/Chunk

Relacionado: `docs/usecases/uc01_ingestao_indexacao_upload_normalize_chunk.md`

### Nós e papéis
- **ReceiveUpload**: Recebe arquivos (upload/ZIP/GDrive) e metadados de sessão/opções.
- **DetectSource**: Identifica `sourceType` e resolve estratégia de leitura.
- **NormalizeDocuments**: Converte para texto estruturado, extrai metadados e valida.
- **ChunkDocuments**: Segmenta texto em chunks com janelas e carrega metadados herdados.
- **PersistChunks**: Persiste chunks e retorna estatísticas/identificadores.

### Arestas e condições
- `ReceiveUpload → DetectSource`: quando `uploadInfo.valid`.
- `DetectSource → NormalizeDocuments`: quando `sourceTypes.length > 0`.
- `NormalizeDocuments → ChunkDocuments`: quando `normalized.valid`.
- `ChunkDocuments → PersistChunks`: quando `chunks.length > 0`.

### Schemas de nós

#### ReceiveUpload
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "sessionId": { "type": "string" },
    "messageExternalId": { "type": "string" },
    "sources": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "sourceType": { "type": "string", "enum": ["upload", "zip", "gdrive"] },
          "uri": { "type": "string" },
          "fileName": { "type": "string" }
        },
        "required": ["sourceType", "uri"]
      }
    },
    "options": {
      "type": "object",
      "properties": {
        "languageHint": { "type": "string" },
        "maxFileSizeMB": { "type": "number" }
      }
    }
  },
  "required": ["sessionId", "messageExternalId", "sources"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "uploadInfo": {
      "type": "object",
      "properties": {
        "valid": { "type": "boolean" },
        "files": { "type": "array", "items": { "type": "string" } },
        "sourceTypes": { "type": "array", "items": { "type": "string" } }
      },
      "required": ["valid", "files", "sourceTypes"]
    }
  },
  "required": ["uploadInfo"]
}
```

#### DetectSource
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "uploadInfo": { "type": "object" }
  },
  "required": ["uploadInfo"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "sourceResolution": {
      "type": "object",
      "properties": {
        "sourceTypes": { "type": "array", "items": { "type": "string" } },
        "strategies": { "type": "array", "items": { "type": "string" } },
        "valid": { "type": "boolean" }
      },
      "required": ["sourceTypes", "strategies", "valid"]
    }
  },
  "required": ["sourceResolution"]
}
```

#### NormalizeDocuments
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "sourceResolution": { "type": "object" },
    "sessionId": { "type": "string" }
  },
  "required": ["sourceResolution", "sessionId"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "normalized": {
      "type": "object",
      "properties": {
        "valid": { "type": "boolean" },
        "documents": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "documentId": { "type": "string" },
              "text": { "type": "string" },
              "metadata": { "type": "object" }
            },
            "required": ["documentId", "text"]
          }
        }
      },
      "required": ["valid", "documents"]
    }
  },
  "required": ["normalized"]
}
```

#### ChunkDocuments
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "normalized": { "type": "object" },
    "chunking": {
      "type": "object",
      "properties": {
        "maxTokens": { "type": "number" },
        "overlapTokens": { "type": "number" }
      },
      "required": ["maxTokens"]
    }
  },
  "required": ["normalized", "chunking"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "chunks": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "chunkId": { "type": "string" },
          "documentId": { "type": "string" },
          "text": { "type": "string" },
          "position": { "type": "number" },
          "metadata": { "type": "object" }
        },
        "required": ["chunkId", "documentId", "text", "position"]
      }
    }
  },
  "required": ["chunks"]
}
```

#### PersistChunks
Input schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "chunks": { "type": "array", "items": { "type": "object" } },
    "sessionId": { "type": "string" }
  },
  "required": ["chunks", "sessionId"]
}
```
Output schema:
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "persistResult": {
      "type": "object",
      "properties": {
        "persisted": { "type": "boolean" },
        "chunkCount": { "type": "number" },
        "storageRefs": { "type": "array", "items": { "type": "string" } }
      },
      "required": ["persisted", "chunkCount"]
    }
  },
  "required": ["persistResult"]
}
```


