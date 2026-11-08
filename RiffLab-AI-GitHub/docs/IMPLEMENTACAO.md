# Plano de implementação e aceitação

## Fase 0 — Verificar ambiente e escolhas
- Criar monorepo `apps/web`, `apps/api`, `packages/shared` (ou estrutura equivalente simples), README e CI.
- Fixar versões e documentar como rodar localmente no Windows e Codespaces.
- Documentar aviso de direitos sobre áudios, letras e fontes externas.
- Criar fixtures de **áudio original/sintético** com pitch e ritmos conhecidos.

**Aceite:** frontend inicial roda; `/api/health` responde; lint/typecheck/testes automatizados passam; CI no GitHub.

## Fase 1 — Interface e fluxo de importação
- UI pt-BR responsiva com campo URL e upload; escolha de afinação e nível; estado de carregamento.
- Parse seguro de links YouTube/Spotify; metadados com integração opcional por chaves configuráveis. Sem extração de áudio.
- Upload de WAV/MP3/FLAC dentro de limite configurável; waveform e recorte de 10–30 segundos.

**Aceite:** URL retorna metadados sem falsa promessa de áudio; upload validado e que excede limite recebe erro claro; testes de URL inválida.

## Fase 2 — Transcrição monofônica real
- Pipeline FFmpeg + librosa + Basic Pitch; saída `NoteEvent` com pitch, início, duração e confiança.
- Separar transcrição real de modo demonstração; nenhum array de notas hardcoded como se fosse resultado de IA.
- Comparar com 5–10 fixtures originais/sintéticas e relatar erros.

**Aceite:** testes com áudio real/sintético fornecido no repositório mostram extração de notas; não há resposta falsa quando modelo indisponível.

## Fase 3 — Tablatura, validação e prévia sonora
- Gerador de candidatos corda/casa, otimizador ergonomia e 2–3 variantes.
- Invariantes mecânicos, reconstrução de pitch, comparação de notas e tempos, síntese em SoundFont compatível e arquivo/stream de prévia.
- Visualizador alphaTab com loop, BPM e destaque de notas; relatório da validação.
- Editar nota/corda/casa reexecuta a validação.

**Aceite:** nenhum `validated` sem `validationDetails` e `previewAudio`; casos impossíveis reprovam, variantes executáveis passam; reproduzir prévia da tab gera as notas previstas.

## Fase 4 — Professor virtual em tempo real
- Captura mic no browser, YIN/autocorrelação ou detector comprovado, calibração de latência/ruído.
- Exercício monofônico sobre tab com marcador de tempo e pontuação decomponível.
- Layout de erro de nota vs erro rítmico, controle de repetição.

**Aceite:** 8–12 testes com tom senoidal e gravações autorizadas verificam equivalência pitch-cents e tempo; ruído não gera acertos; permission denied informa claramente.

## Fase 5 — Harmonia e pesquisa
- Detectar acordes básicos com chroma/Essentia (cientes da qualidade experimental em arranjos complexos).
- Busca de referências externas conforme licenças; registrar proveniência e divergências.
- Recursos de letra apenas quando permitidos por licença.
- Habilitar Demucs opcional para áudio mais complexo e informar custo computacional.

**Aceite:** cada anotação informa se é inferência própria ou referência autorizada; nenhuma letra integral de obra protegida é copiada automaticamente de fonte não licenciada.

## Matriz obrigatória de testes de validação

| Cenário | Esperado |
|---|---|
| Nota MIDI 40 na corda 6, casa 0 em EADGBE | Aceito |
| Nota MIDI 39 sem transposição em EADGBE | Rejeitado (abaixo da extensão) |
| Casa 25 em guitarra configurada com 22 casas | Rejeitado |
| Duas notas diferentes simultâneas na mesma corda | Rejeitado |
| Uma nota em corda e casa que produz outro pitch | Rejeitado |
| Troca irreal de posição em tempo mínimo | Rejeitado ou marcado para revisão, conforme critérios documentados |
| Arquivo com áudio muito ruidoso | Sem selo de validação, indicar incerteza |
| Edição da tablatura validada | Remover validação anterior e revalidar |
| Link Spotify/YouTube sem áudio autorizado | Metadados e player/link permitidos, sem extração |
| Microfone negado | Não quebrar a UI; informar como ativar |
| Nota correta 440 Hz vs referência A4 com margem | Acerto, dependendo da janela temporal |
| Ruído/silêncio | Não gerar nota válida nem nota errada automaticamente |
| Modelo não instalado/erro de inferência | Mensagem de falha real, sem dados inventados |

## Recomendações de desenvolvimento com Copilot

- Trabalhar por PRs pequenos; perguntar ao agente quais arquivos criou e quais testes efetivamente executou.
- Impedir 'mock' de transcrição nos caminhos de produção.
- Colocar bibliotecas pesadas como extras isolados; CI deve rodar testes rápidos com fixtures.
- Registrar o que está 'implementado', 'parcial' e 'planejado' no README, sem exageros.
