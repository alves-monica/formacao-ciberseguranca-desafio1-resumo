<div align="center">

### "Você não precisa de uma pistola de confete para saber que alguém está mentindo." *(adaptado — Patrick Jane)*

`Reconnaissance` · `Active Enumeration` · `Vulnerability Triage`

</div>

<img src="https://capsule-render.vercel.app/api?type=rect&color=B0001E&height=3" width="100%" />

## Sobre este exercício

Laboratório de treino: ambiente intencionalmente vulnerável (Metasploitable 2 + alvo de enumeração SNMP/LDAP), rodado localmente em containers isolados, sem rede exposta à internet.

> IPs, domínios e credenciais abaixo são do ambiente de laboratório local, recriável por qualquer pessoa com os mesmos containers. Nada aqui identifica infraestrutura de terceiros.

O objetivo não foi "rodar ferramentas" — foi treinar a pergunta que separa um scanner de um analista: **diante de 26 portas abertas e 5 serviços enumeráveis, qual é a história real por trás dos dados, e qual é o único fio que, puxado, resolve o caso?**

<img src="https://capsule-render.vercel.app/api?type=rect&color=B0001E&height=3" width="100%" />

## A observação que resolveu o caso

Cinco serviços diferentes (FTP, SMB, SNMP, LDAP, distcc) devolveram informação sem exigir prova de identidade de quem perguntou. Não são cinco problemas — é **um problema só**, repetido em cinco fantasias:

> *"Cavalheiros, vocês não têm cinco suspeitos. Têm a mesma impressão digital em cinco cenas de crime."*

| Serviço | O que respondeu sem autenticação |
|---|---|
| SMB | Sessão nula aceita — 36 contas de sistema enumeradas, política de senha sem complexidade mínima |
| SNMP | Community string de fábrica (`public`), sem lista de controle de origem |
| LDAP | Bind anônimo — estrutura organizacional completa, com identidade de pessoas (nome, cargo, e-mail) |
| FTP | Versão com backdoor conhecido publicamente — execução de comando confirmada |
| distcc | Serviço desenhado para aceitar e executar comando de qualquer origem na rede |

Tratar isso como "5 vulnerabilidades" no relatório seria perder o ponto. Tratar como "1 postura de segurança (confiança implícita em serviço de rede) manifestada 5 vezes" é o que de fato importa para quem vai corrigir — porque a correção é a mesma receita, repetida 5 vezes, não 5 receitas diferentes.

<img src="https://capsule-render.vercel.app/api?type=rect&color=B0001E&height=3" width="100%" />

## Metodologia (a ordem importa)

```text
RECONHECIMENTO PASSIVO  (o alvo não percebe)
        ↓
VARREDURA ATIVA         (nmap — quais portas respondem?)
        ↓
ENUMERAÇÃO              (o que cada serviço conta, sem pedir senha?)
        ↓
CONFIRMAÇÃO             (a falha teórica é real neste host, agora?)
        ↓
TRIAGEM                 (qual achado, entre muitos, decide a prioridade?)
```

Cada etapa só avança com o que a anterior confirmou — um achado de "superfície de ataque" sem lastro na etapa de enumeração é especulação, não achado.

### Achados confirmados (não apenas apontados pelo scanner)

| # | Achado | Confirmação | Severidade |
|---|---|---|---|
| 1 | Backdoor em versão pública de servidor FTP | Execução de comando confirmada via exploit público — acesso de root | Crítica |
| 2 | Daemon de compilação distribuída aceitando execução remota | Execução de comando confirmada — acesso de serviço | Crítica |
| 3 | Diretório corporativo acessível sem autenticação | Estrutura e identidade de 4 pessoas extraídas sem credencial | Alta |

### O que foi testado e descartado (e por quê isso entra no relatório)

Uma falha clássica de SMB em Windows foi testada e **não** se confirmou — o serviço aqui era outra implementação, em outro sistema operacional. Isso entra no relatório com o mesmo peso de um achado positivo: prova que a varredura foi deliberada, cobrindo hipóteses, não um recorte dos resultados convenientes.

> *"Eu não acuso ninguém até ter certeza. Prefiro descartar dez suspeitos a acusar o errado."*

### O que foi identificado e **não** executado

Uma vulnerabilidade de negação de serviço apareceu no scanner. Não foi acionada — ambiente de treino ou não, provocar indisponibilidade deliberada não faz parte do exercício de reconhecimento, e a regra vale em qualquer engajamento real: autorização para *encontrar* uma falha não é autorização automática para *acioná-la* quando o efeito é indisponibilidade de terceiros.

<img src="https://capsule-render.vercel.app/api?type=rect&color=B0001E&height=3" width="100%" />

## O raciocínio, não só o resultado

A parte que mais valeu a pena documentar não foi a lista de portas abertas — foi **por que** cada decisão de priorização foi tomada:

- **Exploração confirmada pesa mais que CVSS alto isolado.** Dois achados tinham CVSS crítico; um deles devolveu root de fato, o outro devolveu um usuário de serviço. A ordem de prioridade reflete essa diferença, não só o número do scanner.
- **Vazamento de identidade de pessoa muda a categoria do risco.** Um achado técnico (versão de sistema, processo rodando) e um achado de identidade (nome + cargo + e-mail de uma pessoa real) não competem na mesma escala — o segundo habilita engenharia social, o primeiro não.
- **O mapeamento de tática/técnica segue o dado, não a ferramenta.** A mesma ferramenta de enumeração, rodada duas vezes contra alvos diferentes, pode render duas táticas diferentes, dependendo do que efetivamente saiu.

<img src="https://capsule-render.vercel.app/api?type=rect&color=B0001E&height=3" width="100%" />

## Lições que ficam

1. Cinco achados com a mesma causa raiz contam uma história melhor que cinco achados isolados.
2. "O scanner marcou como vulnerável" e "eu confirmei que é explorável agora" são afirmações diferentes — só a segunda vai para o topo da priorização.
3. Testar uma hipótese e descartá-la tem valor documental — mostra rigor, não ausência de achado.
4. CVSS é ponto de partida da priorização, não o fim dela — o contexto (o que o achado habilita a seguir) decide o resto.
___