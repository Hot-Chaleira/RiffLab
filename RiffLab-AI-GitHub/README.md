# RiffLab AI — kit de planejamento para desenvolvimento com GitHub Copilot

> **Estado:** documentação de produto e desenvolvimento; não é um aplicativo funcional.
>
> **Objetivo:** professor virtual de guitarra que pesquisa música, transcreve áudio autorizado, gera cifras e tablaturas, valida a executabilidade, sintetiza a versão proposta e acompanha a execução pelo microfone.

## Como começar no GitHub

1. Crie um repositório **privado** chamado `rifflab-ai` no GitHub.
2. Faça upload dos arquivos deste kit, preservando as pastas `.github/` e `docs/`. É necessário **extrair o ZIP** antes do upload.
3. Abra o repositório no VS Code, GitHub Codespaces ou ambiente que tenha o GitHub Copilot Chat em **modo agente**.
4. Abra `PROMPT_COPILOT.md`, copie seu conteúdo e envie ao Copilot. Peça **apenas a primeira fase** e exija que rode testes antes de continuar.
5. Teste localmente o protótipo e avance etapa por etapa conforme `docs/IMPLEMENTACAO.md`.

## Arquivos

- `PROMPT_COPILOT.md`: prompt inicial para criar o projeto com o agente Copilot.
- `.github/copilot-instructions.md`: regras persistentes para o Copilot.
- `docs/ESPECIFICACAO.md`: produto, arquitetura, funcionalidades, contrato de dados e validação obrigatória.
- `docs/IMPLEMENTACAO.md`: etapas, testes e critérios de aceite.

## Escopo técnico inicial

- **Frontend:** React + TypeScript + Vite + CSS responsivo.
- **Backend:** FastAPI + Python; tarefas longas assíncronas de processamento, com status claro.
- **Processamento:** FFmpeg (áudio autorizado), librosa, Basic Pitch; Demucs opcional (mais pesado).
- **Tablatura:** alphaTab, usando seu modelo de dados ou MusicXML/alphaTex compatíveis.
- **Treino:** Web Audio API, getUserMedia e AudioWorklet; algoritmo monofônico YIN/autocorrelação para riffs isolados.
- **Busca:** YouTube Data API, Spotify Web API, MusicBrainz para **metadados**, fontes de notação apenas quando autorizadas.
- **Letras:** somente conteúdo fornecido pelo usuário ou serviço contratado/licenciado.
- **Persistência:** SQLite no desenvolvimento, migração opcional para PostgreSQL.

## Restrições importantes

- Links do Spotify e YouTube **não autorizam** extrair e analisar seus streams. Não implemente download clandestino, scraping de mídia, captura de áudio do player ou desvio de DRM.
- Não afirme que uma tablatura ficou 'bonita' só porque notas/frequências batem. A validação de áudio é **objetiva e parcial**; indique limitação musical e permita audição humana.
- A análise de gravações completas com múltiplos instrumentos pode falhar; inicialmente priorize riffs, melodias monofônicas e arquivos autorizados.
- Uma tablatura só é liberada quando passou por checagens de tessitura, atribuição corda/casa, ergonomia e comparação da síntese com as notas-alvo. Se não passar, marcar 'revisão necessária'.
- GitHub Pages hospeda somente a parte estática; a análise em Python requer servidor ou serviço computacional separado.

## Referências para documentação oficial

- Basic Pitch: https://github.com/spotify/basic-pitch
- Demucs: https://github.com/facebookresearch/demucs (repositório original arquivado; avaliar compatibilidade)
- alphaTab: https://github.com/CoderLine/alphaTab
- Essentia: https://essentia.upf.edu/
- MusicBrainz API: https://musicbrainz.org/doc/MusicBrainz_API
- YouTube Developer Policies: https://developers.google.com/youtube/terms/developer-policies
- Spotify Developer Policy: https://developer.spotify.com/policy
- GitHub Copilot instructions: https://docs.github.com/en/copilot/concepts/prompting-and-context/custom-instructions
