# NetTrap — Plataforma de Honeypots com IA (NIS2)

> **Sistema de deteção e resposta a ameaças para PMEs**, com 7 honeypots
> multi-protocolo, análise comportamental por IA e conformidade com a
> Diretiva NIS2. Desenvolvido na **Expertmode**.

![status](https://img.shields.io/badge/status-em%20produção-success)
![python](https://img.shields.io/badge/Python-3.12-blue)
![NIS2](https://img.shields.io/badge/NIS2-Article%2023-orange)
![license](https://img.shields.io/badge/código-privado-lightgrey)

> ℹ️ Este repositório é uma **vitrine técnica**. O código-fonte é
> propriedade da Expertmode e permanece privado. Aqui descrevo a
> arquitetura, as decisões técnicas e as funcionalidades — sem expor
> implementação, lógica de deteção ou configuração.

---

## O problema

A Diretiva **NIS2** obriga milhares de PMEs europeias a detetar e
notificar incidentes de cibersegurança em prazos apertados (alerta
precoce em 24h, notificação em 72h — Artigo 23). A maioria não tem SOC,
nem orçamento para soluções enterprise (€7.500+/ano por dispositivo).

O **NetTrap** resolve isto: um *appliance* acessível que atrai atacantes
para iscos (honeypots), analisa o comportamento com IA, bloqueia
automaticamente e gera a documentação de conformidade NIS2.

---

## Demonstração

<!-- Substitui estas imagens pelas tuas screenshots (só com dados de
     demonstração: sem IPs reais, sem credenciais, sem dados de clientes).
     Coloca os PNG em docs/img/ com estes nomes. -->

| Dashboard em tempo real | Relatório de conformidade NIS2 |
|:---:|:---:|
| ![Dashboard](docs/img/dashboard.png) | ![Relatório NIS2](docs/img/nis2-report.png) |

| Gestão de alertas | Perfil & MFA |
|:---:|:---:|
| ![Alertas](docs/img/alerts.png) | ![MFA](docs/img/mfa.png) |

---

## Arquitetura (visão geral)

```
                       ┌─────────────────────────┐
   Atacante  ──────►   │   7 Honeypots multi-     │
                       │   protocolo (iscos)      │
                       │ SSH HTTP SMB FTP Telnet  │
                       │ MySQL RDP + canary       │
                       └────────────┬─────────────┘
                                    │ eventos
                       ┌────────────▼─────────────┐
                       │   Pipeline de eventos     │
                       │  (memória + persistência) │
                       └────────────┬─────────────┘
              ┌─────────────────────┼─────────────────────┐
   ┌──────────▼─────────┐ ┌─────────▼────────┐ ┌──────────▼─────────┐
   │  5 Agentes de IA   │ │   Alertas         │ │  Conformidade NIS2 │
   │ Scanner Profiler   │ │  Email + MQTT     │ │  Notificação CERT  │
   │ Deceiver Resolver  │ │                   │ │  Audit trail       │
   │ Maestro            │ │                   │ │  Retenção          │
   └────────────────────┘ └───────────────────┘ └────────────────────┘
                                    │
                       ┌────────────▼─────────────┐
                       │  Dashboard HTTPS (web)    │
                       │  JWT · RBAC · MFA TOTP    │
                       └──────────────────────────┘
```

### Os 5 agentes de IA
| Agente | Função |
|--------|--------|
| **Scanner** | Classifica a ameaça (tipo de ataque, severidade) a partir de features comportamentais |
| **Profiler** | Mapeia técnicas MITRE ATT&CK e deteta anomalias face a um baseline |
| **Deceiver** | Gera respostas credíveis para manter o atacante envolvido |
| **Maestro** | Orquestra a pipeline e decide a ação (envolver / bloquear) |
| **Resolver** | Aplica bloqueio automático (firewall + blocklist partilhada) |

---

## Funcionalidades principais

- 🪤 **7 honeypots multi-protocolo** + canary tokens
- 🧠 **Análise comportamental por IA** (5 agentes coordenados)
- 🚫 **Bloqueio automático** de atacantes
- 📊 **Dashboard web em tempo real** (WebSocket) sobre **HTTPS**
- 🔐 **Segurança**: autenticação JWT, RBAC (admin/analista/viewer),
  **MFA TOTP**, cabeçalhos CSP/HSTS, *secrets* encriptados (Fernet)
- 📁 **Persistência fiável** (SQLite + cache em memória), **backups
  encriptados automáticos**
- 🇪🇺 **Conformidade NIS2**: notificação automática à CSIRT (Artigo 23),
  *audit trail* com cadeia de hash **verificada** (tamper-evident),
  política de retenção
- 📄 **Relatórios** em formato STIX 2.1 para submissão à CERT.PT

---

## Stack técnica

**Backend:** Python 3.12 · FastAPI · asyncio · SQLite (aiosqlite) · ZeroMQ
**Frontend:** HTMX · Alpine.js · Tailwind CSS · WebSocket
**IA/ML:** ONNX Runtime · scikit-learn · NumPy · mapeamento MITRE ATT&CK
**Segurança:** JWT (python-jose) · bcrypt · pyotp (TOTP) · cryptography (Fernet)
**Infra:** Alpine Linux · OpenRC · Docker · nftables · fail2ban
**Qualidade:** pytest (suite de regressão de segurança) · CI (GitHub Actions)

---

## Engenharia & decisões técnicas

Alguns dos desafios que resolvi neste projeto:

- **Pipeline de eventos resiliente** — *fan-out* de cada evento para 5
  destinos (memória, base de dados, alertas, IA, conformidade) sem
  perder eventos forenses, com *timeouts* e *logging* de falhas.
- **Tamper-evidence real** — implementei verificação periódica da cadeia
  de *hash* Merkle do *audit log*; deteta adulteração da base de dados
  (requisito NIS2 que muitas soluções só simulam).
- **Hardening de segurança** — auditoria interna que identificou e
  corrigiu XSS armazenado, CORS aberto, ausência de *security headers* e
  *secrets* por omissão; tudo coberto por testes de regressão.
- **Instalador *one-command*** para Alpine (de SSH manual para um único
  comando reproduzível) e empacotamento para *appliance* físico ou VM.

---

## Modelos de implementação

| Tier | Formato | Caso de uso |
|------|---------|-------------|
| **Pro** | Appliance físico pré-configurado (mini-PC) | PME que quer chave-na-mão |
| **Standard** | ISO/OVA para a VM do cliente | Quem já tem virtualização |
| **Cloud** | Instância gerida | Honeypot exposto à internet |

---

## O meu papel

Desenvolvimento da plataforma na Expertmode: arquitetura do *backend*,
pipeline de eventos, integração dos agentes de IA, *dashboard*,
conformidade NIS2, *hardening* de segurança, testes e empacotamento/deploy.

> 📬 Detalhes técnicos adicionais ou demonstração mediante pedido.
> O código-fonte é privado (propriedade da Expertmode).
