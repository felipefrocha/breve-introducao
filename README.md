# Uma breve introdução

## Propósito
Nivelar o conhecimento em diversas ferramentas e indicar material de leitura para diversos temas.

## Temas
### 1. Docker
#### 1.1 Instalação
```bash
# Processo como root (not recommended)
curl -fsSL https://get.docker.com | sudo bash -
# Processo como rootless -> veja a documentação https://docs.docker.com/engine/security/rootless/
curl -fsSL https://get.docker.com/rootless | sh
curl -L https://raw.githubusercontent.com/docker/compose-cli/main/scripts/install/install_linux.sh | sh
```
#### 1.2 Links
- [Docker e VS Code](https://code.visualstudio.com/docs/remote/containers-tutorial)
- [Arquitetura de containers]()
- [Segurança de contaienrs]()
- [Containers RedHat](https://redhatgov.io/workshops/containers_101/)

#### 1.3 Comunicação Segura
- TLS:
  ```bash
  # Criar CA
  openssl genrsa -aes256 -out ca-key.pem 4096
  openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem
  # Certificado chave Servidor 
  openssl genrsa -out server-key.pem 4096
  openssl req -subj "/CN=$HOST" -sha256 -new -key server-key.pem -out server.csr
  ## Configuração do docker daemon
  echo subjectAltName = DNS:$HOST,IP:10.10.10.20,IP:127.0.0.1 >> extfile.cnf
  echo extendedKeyUsage = serverAuth >> extfile.cnf
  # Cria certificado Auto assinado do servidor
  openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem \
  -CAcreateserial -out server-cert.pem -extfile extfile.cnf
  
  # Chave do Cliente
  openssl genrsa -out key.pem 4096
  # Assinatura do cliente
  openssl req -subj '/CN=client' -new -key key.pem -out client.csr
  ## Configuração do cliente
  echo extendedKeyUsage = clientAuth > extfile-client.cnf
  # Certificado para Cliente
  openssl x509 -req -days 365 -sha256 -in client.csr -CA ca.pem -CAkey ca-key.pem \
  -CAcreateserial -out cert.pem -extfile extfile-client.cnf
  
  
  # Limpa arquivos 
  rm -v client.csr server.csr extfile.cnf extfile-client.cnf
  # Permissão dos arquivos
  chmod -v 0444 ca.pem server-cert.pem cert.pem
  
  # Configurando Docker Daemon (Server) para aceitar apenas conexão com certificado
  dockerd \
    --tlsverify \
    --tlscacert=ca.pem \
    --tlscert=server-cert.pem \
    --tlskey=server-key.pem \
    -H=0.0.0.0:2376
    
  # Cliente do docker com certificados
  docker --tlsverify \
    --tlscacert=ca.pem \
    --tlscert=cert.pem \
    --tlskey=key.pem \
    -H=$HOST:2376 version
  ```
