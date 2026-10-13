# Cegonha

Aplicativo móvel para gerenciamento de encomendas em condomínios, desenvolvido com React Native, Expo e Firebase.

## Proposta

O Cegonha organiza o ciclo de uma encomenda entre morador e portaria, reduz erros de identificação com OCR, mantém rastreabilidade das ações e centraliza avisos.

## Fluxo da encomenda

Uma encomenda pode apresentar a linha do tempo:

```text
Prevista pelo morador
        ↓
Recebida pela portaria
        ↓
Aguardando retirada
        ↓
Retirada
```

Cada evento guarda data, horário e responsável quando aplicável.

## OCR com confirmação humana

O OCR usa OCR.Space e nunca registra uma encomenda sozinho. Depois da leitura, o app mostra candidatos com percentual de compatibilidade. A portaria confirma ou corrige o destinatário.

A tela também registra condições para avaliação acadêmica:

- texto impresso ou manuscrito;
- iluminação boa, média ou ruim;
- tempo de processamento;
- campos reconhecidos;
- primeiro candidato sugerido;
- candidato confirmado;
- acerto do primeiro candidato.

Os resultados ficam em `ocrTestes` e aparecem no relatório do condomínio.

Veja o protocolo em [docs/AVALIACAO_OCR.md](docs/AVALIACAO_OCR.md).

## Prevenção de duplicidade

Ao registrar uma nova chegada, o aplicativo verifica se o mesmo morador recebeu outra encomenda nos últimos 10 minutos. Quando encontra uma possível duplicidade, a portaria precisa confirmar antes de continuar.

## Vínculo de moradores

O morador pode criar a conta sem possuir o código do condomínio.

Quando receber o código:

1. informa o código no Perfil;
2. envia uma solicitação de entrada;
3. a portaria aprova ou recusa;
4. o morador recebe uma notificação;
5. após aprovação, conclui o vínculo pelo Perfil.

## Dashboard da portaria

O painel mostra:

- encomendas recebidas hoje;
- encomendas aguardando retirada;
- retiradas de hoje;
- encomendas há mais de 3 dias;
- tempo médio de retirada;
- moradores com mais encomendas pendentes;
- solicitações de entrada;
- acesso aos indicadores do OCR.

Pendências com mais de 3 dias podem gerar um lembrete interno ao morador.

## Histórico e auditoria

Além da linha do tempo salva na encomenda, eventos relevantes são registrados na coleção `auditoria`, incluindo chegada, retirada e envio de lembrete.

## Busca e filtros

A portaria pode buscar por nome, bloco, apartamento, descrição ou código da encomenda e filtrar por:

- Pendentes;
- Hoje;
- +3 dias;
- Retiradas;
- Todas.

O morador também possui busca e filtros no próprio histórico.

## Notificações

A central interna diferencia notificações lidas e não lidas, permite marcar uma individualmente ou todas de uma vez e possui eventos de:

- chegada;
- retirada;
- lembrete;
- vínculo com condomínio.

## Detalhes da encomenda

A tela de detalhes mostra:

- destinatário;
- bloco e apartamento;
- status;
- código de referência;
- miniatura da etiqueta quando disponível;
- portaria responsável;
- texto reconhecido pelo OCR;
- linha do tempo;
- registro da retirada.

## Perfil

O perfil permite:

- foto;
- dados do usuário;
- bloco/apartamento para morador;
- dados do condomínio;
- solicitação de vínculo;
- cópia do código para portaria;
- acesso à tela de privacidade.

## Design e acessibilidade

- ícones vetoriais simples com Ionicons;
- nenhuma ação principal depende de emoji;
- menu inferior flutuante acima da área de navegação do aparelho;
- estados vazios e skeletons de carregamento;
- cabeçalhos nativos removidos;
- **EB Garamond** usada em todos os textos, títulos, botões, campos e rótulos do aplicativo;
- o nome **Cegonha** também usa EB Garamond em peso 700;
- botões e áreas de toque possuem tamanhos maiores e contraste alto.

## Onboarding

No primeiro acesso são exibidas três etapas:

1. **Receba**
2. **Identifique**
3. **Avise**

A conclusão é guardada no AsyncStorage.

## Privacidade

O app possui uma tela própria explicando os dados usados e o processamento externo do OCR.

As imagens de etiquetas são enviadas ao OCR.Space. Após uma chegada ser confirmada, o app pode salvar uma miniatura comprimida separadamente em `fotosEncomendas`.

## Coleções principais

```text
usuarios
condominios
solicitacoesCondominio
encomendas
fotosEncomendas
previsoes
notificacoes
ocrTestes
auditoria
fotosPerfil
```

## Instalação

```bash
git pull origin main
npm install
npx expo install --fix
npx expo start -c
```

Nesta atualização, `npm install` continua necessário para garantir as versões atuais do Expo AuthSession, EB Garamond e demais dependências.

## Regras do Firestore

Depois de atualizar o projeto:

```bash
npm run firebase:rules
```

As novas coleções de solicitações, auditoria, métricas de OCR e fotos de encomendas dependem das regras atuais do repositório.

## Testes

O plano de testes do TCC está em:

[docs/TESTES.md](docs/TESTES.md)

Ele contém 30 cenários cobrindo cadastro, perfis, vínculo, OCR, duplicidade, histórico, filtros, notificações, segurança e usabilidade.

## Login com Google

O login/cadastro com Google está integrado ao Firebase Authentication por `expo-auth-session`. O botão aparece no Login e no Cadastro.

No `.env`:

```env
EXPO_PUBLIC_GOOGLE_WEB_CLIENT_ID=
EXPO_PUBLIC_GOOGLE_ANDROID_CLIENT_ID=
EXPO_PUBLIC_GOOGLE_IOS_CLIENT_ID=
```

Também é necessário ativar **Google** em `Firebase Authentication > Sign-in method`. Sem um Client ID válido, o aplicativo mostra uma mensagem de configuração em vez de falhar silenciosamente.

## Entregas previstas

O morador pode criar, **editar** e **excluir** entregas previstas. Se uma previsão for vinculada a uma chegada pela portaria, os dados relevantes já ficam copiados para a linha do tempo da encomenda.

## Notificações de chegada

O registro de uma encomenda e a criação da notificação do morador são feitos no mesmo batch do Firestore. Assim, a chegada só é concluída quando o aviso também é gravado. A Central de Notificações usa atualização em tempo real com `onSnapshot`.

## Vínculo por código

O vínculo do morador foi padronizado:

1. o morador informa o código;
2. o código é normalizado e validado na coleção `condominios`;
3. uma solicitação é criada para o condomínio correto;
4. a portaria aprova;
5. o morador conclui o vínculo;
6. as regras do Firestore verificam a solicitação aprovada antes de permitir a troca de `condominioId`.

Solicitações rejeitadas podem ser reenviadas sem quebrar o fluxo.

## Teclado e formulários

As telas de Login, Cadastro, Recuperação de senha, Perfil, Entregas previstas e Registro de chegada usam um contêiner de teclado com rolagem. No Android, `softwareKeyboardLayoutMode: "resize"` continua ativo para impedir que o teclado cubra o campo em edição.

## OCR

Configure no `.env`:

```env
EXPO_PUBLIC_OCR_SPACE_API_KEY=SUA_CHAVE
```

Para testes, o projeto mantém o fallback `helloworld`, sujeito aos limites do OCR.Space.

O OCR não usa Firebase Cloud Functions e não precisa de plano Blaze.
