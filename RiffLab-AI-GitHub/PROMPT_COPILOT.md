# Prompt de abertura para colar no GitHub Copilot Chat (modo agente)

Você é engenheiro de software sênior full-stack com experiência em processamento digital de sinais, transcrição musical automática e pedagogia de guitarra. Vamos construir o **RiffLab AI**, uma aplicação web, em português brasileiro, que me permita aprender guitarra Stratocaster.

Leia primeiro `docs/ESPECIFICACAO.md`, `docs/IMPLEMENTACAO.md` e `.github/copilot-instructions.md` e siga todas as restrições. **Não tente construir tudo de uma vez.** Quero um projeto funcional e evolutivo, não uma interface bonita com botões falsos.

### Produto final

1. Pesquisar música por título, artista ou link Spotify/YouTube e obter metadados e referências autorizadas; nunca extrair stream sem licença.
2. Aceitar upload autorizado de áudio e selecionar trecho (riff principal, melodia, solo, harmonia).
3. Usar análise real de áudio para detectar BPM, notas, ataques e, mais tarde, acordes e separação de instrumentos.
4. Converter notas em alternativas reais de tablatura e cifras; mostrar número da corda e casa, sugestões de digitação e níveis de dificuldade.
5. **Antes de mostrar a tablatura como validada** executar verificação estrutural/ergonômica, reconstituir as alturas da tablatura, gerar uma prévia de guitarra sintetizada e comparar as notas e ritmos sintetizados com o alvo. Exigir status honesto `validated`, `review_required` ou `rejected` e razões detalhadas. Não presumir que pontuação técnica garanta beleza artística.
6. Exibir tablatura interativa via alphaTab, permitir ouvir a prévia, reduzir velocidade, repetir trecho, usar metrônomo e editar posições.
7. Permitir tocar diante do microfone, com calibração de ruído e latência, detectar notas de riff monofônico e mostrar acertos e erros de altura e tempo em tempo real.
8. Exibir acordes com letras apenas quando houver conteúdo enviado pelo usuário ou licenciado.

### Arquitetura preferida

Frontend React + TypeScript + Vite; backend Python + FastAPI; SQLite; Basic Pitch, librosa, FFmpeg, alphaTab; Demucs opcional; Web Audio API + AudioWorklet para microfone. Trabalhar em CPU, pensando em computadores sem GPU dedicada. Crie modelos de dados e testes conforme a especificação.

### O que fazer AGORA: apenas Fase 0 e Fase 1

- Crie estrutura do monorepo, dependências, README, scripts de execução, arquivos `.env.example`, CI GitHub Actions, lint, typecheck e testes.
- Implemente interface responsiva em pt-BR para busca por URL ou nome da música, upload autorizado MP3/WAV/FLAC, afinação padrão EADGBE, nível e opção de trecho; indicação honesta das limitações.
- Implemente `GET /api/health` e `POST /api/sources/resolve` com validação de URL, rotas de metadados que funcionem sem credencial em modo limitado e integração opcional com APIs oficiais mediante chaves. Nunca baixar áudio de Spotify ou YouTube.
- Implemente upload com arquivo salvo temporariamente com limites, interface de seleção de trecho e waveform real caso viável no escopo da fase.
- Crie testes unitários/integração, execute-os, corrija os erros e documente comandos precisos para Windows/VS Code e Codespaces.
- Liste separadamente recursos operacionais e recursos ainda não implementados. Não coloque botões operacionais de transcrição/treino que não funcionam; use rótulo desabilitado 'Em desenvolvimento' quando necessário.

Ao terminar, mostre árvore de pastas, instruções para executar, testes de verdade e o PR/commit que corresponde a esta etapa. **Pare após a Fase 1** para que eu teste antes de pedir a Fase 2.
