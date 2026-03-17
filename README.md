# MFA Flask Prototype (Password + TOTP)

Protótipo simples de **Autenticação Multi‑Fator (MFA)** com **Flask** em Python:
- 1.º fator: *username + password*
- 2.º fator: **TOTP** (compatível com Google Authenticator, Authy, etc.)

## Team Members
Este projeto foi desenvolvido em colaboração por:
* Miguel Bento (GitHub: mvbento)
* Luís Lourenço (GitHub: lcmlo)
* Ricardo Ourelo (GitHub: ricardo-ourelo)
* Josué da Glória (GitHub: josueclm)

## Documentação
O relatório completo com a fundamentação teórica, análise de riscos e revisão de literatura encontra-se no ficheiro: [Secure_Access_SRSI_Theoretical.pdf](./Secure_Access_SRSI_Theoretical.pdf)

## 1) Pré‑requisitos
- Python 3.11+
- (Opcional) Ambiente virtual `venv`

## 2) Instalação (com o terminal na pasta raiz do projeto)
```bash
python -m venv .venv
# Windows:
.venv\Scripts\activate
# Linux/Mac:
source .venv/bin/activate

pip install -r requirements.txt
```

## 3) Variáveis de ambiente (via arquivo .env)
As variáveis são carregadas automaticamente pelo python-dotenv.
Cria um ficheiro chamado .env na raiz do projeto:
```bash
FLASK_SECRET_KEY=uma-chave-forte-aqui
MFA_ENC_KEY=A_TUA_CHAVE_GERADA_AQUI
MFA_DB_PATH=mfa.db
```

Como gerar a chave MFA_ENC_KEY:
```bash
python
from cryptography.fernet import Fernet
print(Fernet.generate_key().decode())
exit()
```

## 4) Executar
Caso ainda não esteja ativa, ativa a venv
```bash
# Windows:
.venv\Scripts\activate
# Linux/Mac:
source .venv/bin/activate
```
Depois, executa a app
```bash
python app.py
# A aplicação ficará disponível em http://127.0.0.1:5000 e como host no seu ip no porto 5000
```

## 5) Fluxo de utilização
1. Acede a **/register** e cria um utilizador.
2. Faz **login** com username/password.
3. Serás redirecionado para **/setup_mfa** (primeiro login):
   - Digitaliza o QR Code com o **Google Authenticator** (ou app semelhante).
   - Introduz o código de 6 dígitos para confirmar.
4. Em logins futuros, o sistema pedirá o **código TOTP** em **/mfa**.
5. Após a validação, serás redirecionado para o **dashboard**.

## 6) Estrutura
```text
mfa_flask_prototype/
├─ .env
├─ app.py
├─ requirements.txt
├─ README.md
├─ templates/
│  ├─ base.html
│  ├─ login.html
│  ├─ register.html
│  ├─ setup_mfa.html
│  ├─ mfa.html
│  └─ dashboard.html
└─ static/
   └─ styles.css
```

## 7) Notas de segurança (produção)
- **NÃO** reutilizes a `SECRET_KEY` deste repositório; define uma chave forte via variável de ambiente.
- Passwords são guardadas com hash (`werkzeug.security`).
- Segredos TOTP encriptados via cryptography.Fernet (AES-128)
- Ativa HTTPS (TLS) quando publicares.
- Considera rate‑limiting e CSRF.
- Garante que o ficheiro .env está incluído no .gitignore, o que vem por default deve ser alterado e excluido de versionamento
- Sessões Flask isoladas por SECRET_KEY
- Para *phishing-resistance*, considera **FIDO2/WebAuthn** (chaves físicas).

## 8) Testes rápidos
- Introduz códigos TOTP errados: o login deve falhar.
- Teste 1: Registos simultâneos com o mesmo username
Um cliente deve conseguir registar
O outro deve receber "Esse username já existe"
- Teste 2: Dois utilizadores ativam MFA ao mesmo tempo -> QR Codes diferentes -> Segredos TOTP diferentes -> Códigos TOTP distintos mesmo no mesmo instante
- Teste 3: Segredos TOTP encriptados ->
BD não deve ter segredos em plaintext ->
Campo totp_secret contém textos iniciados por gAAAA...

## 9) Licença
Uso académico/demonstração.


