# Especificação de Produto — RiffLab AI

## 1. Visão

Uma aplicação web de estudo de guitarra (foco inicial: Stratocaster, seis cordas, afinação padrão E2 A2 D3 G3 B3 E4) que transforma áudio de fonte autorizada em transcrição editável e tocável e que acompanha a execução do aprendiz. A experiência deve ser acessível a iniciantes, em português do Brasil e utilizável em computador e celular.

**Princípio inegociável:** nunca entregar uma tablatura gerada automaticamente como 'verificada' sem executar a cadeia de validação técnica, renderizar uma prévia sonora sintetizada e medir sua correspondência com a interpretação-alvo. Quando a qualidade for insuficiente, não falsificar a aprovação: apresentar 'revisão necessária', opções alternativas ou permitir correção manual.

## 2. Jornada do usuário

1. Informar título e artista, link do Spotify ou YouTube, ou enviar arquivo de áudio sobre o qual tenha autorização de uso.
2. Por URL: identificar a música usando metadados e pesquisar referências disponíveis legalmente. Oferecer player embutido ou link externo autorizado. Explicar que o stream não pode ser analisado só pelo URL.
3. Por arquivo: validar MIME/extensão, tamanho, duração e autorização indicada pelo usuário; gerar waveform e metadados técnicos; normalizar para processamento sem destruir original.
4. Usuário escolhe 'Riff principal', 'Melodia', 'Solo', 'Acordes/cifras' ou 'Música inteira' (este último apenas quando a qualidade permitir). Escolhe nível Iniciante/Intermediário/Original.
5. Processar o áudio em tarefas com progresso real: pré-processamento, estimativa de andamento/tom, separação de fontes opcional, transcrição de notas, acordes e ritmos, mapeamento para as cordas e avaliação.
6. Produzir múltiplos dedilhados/caminhos para as mesmas notas; pontuar dificuldade, fidelidade e ergonomia.
7. **Obrigatório antes da liberação:** testar intervalos de notas, casas, simultaneidade de cordas, sequências de movimentos, ritmos e duração; sintetizar a tablatura e compará-la com o alvo musical. Uma prévia de áudio deve estar disponível ao usuário.
8. Mostrar tablatura rolável e sincronizada, acordes/diagramas e, se houver direito de exibição, letra alinhada. Botões para ouvir exemplo, tocar a 50–100%, repetir trecho, metrônomo, mudar afinação e corrigir tab.
9. Na aba 'Praticar', solicitar autorização de microfone, calibrar ruído/latência e monitorar notas e tempo. Fornecer feedback visual por nota e um resumo ao final sem inventar precisão.
10. Salvar progresso local, última seção e versão editada; permitir exportação MIDI/MusicXML/PDF quando técnica e legalmente permitida.

## 3. Funcionalidades com prioridade

### P0 (produto mínimo genuinamente funcional)
- Entrada de MP3/WAV/FLAC autorizados e metadados via URLs; distinção clara entre as duas capacidades.
- Transcrição **monofônica** de excertos selecionados usando Basic Pitch, com saída de eventos de nota com tempo de início/duração/confiança.
- Construção de 2–3 alternativas de tablatura para guitarra de seis cordas, usando algoritmo de otimização por programação dinâmica.
- Validação mecânica obrigatória, síntese prévia e comparação objetiva com a transcrição-alvo; recusa de publicação como 'validada' em caso de reprovação.
- Leitor interativo de tablatura com execução sintetizada, laço de repetição, alteração de velocidade e deslocamento da região visível.
- Treino monofônico ao vivo com microfone: notas corretas, erros de afinação, omissões e atraso/adiantamento com tolerâncias configuráveis e calibração.
- Edição manual das notas e cordas/casas com revalidação obrigatória.
- Funcionamento local sem conta e com armazenamento local mínimo.

### P1
- Reconhecimento de acordes/cifras por cromagrama (librosa/Essentia), sugerindo tonalidade e mudanças harmônicas com indicadores de confiança.
- Modelo opcional de separação de instrumentos (Demucs) para gravações completas com avisos de artefatos/contaminação.
- Comparação com referências externas de tablatura via APIs/fontes permitidas (nunca scraping indiscriminado), exibindo origem e divergências.
- Diagramas de acordes e editor de cifras alinhadas com os tempos musicais.
- Biblioteca pessoal e histórico de estudo.
- Exportações adicionais (PDF/Guitar Pro conforme bibliotecas compatíveis).

### P2
- Identificação de batidas, arpejos e acordes polifônicos do usuário; avaliação de dinâmica, técnicas (slide/bend/hammer-on/pull-off) com validação própria.
- Conteúdo de letra completa **apenas** com serviço licenciado e autorização de exibição, ou letras fornecidas pelo usuário com direitos adequados.
- Recomendação de exercícios a partir dos erros, progressão de dificuldade, modos guitarra solo/ritmo.
- Sincronização explícita com players externos somente quando permitida por termos e direitos, não assumir que é autorizada.

## 4. Arquitetura

```
[React/Vite/TypeScript]
   |  ├── busca e metadados (YouTube/Spotify/MusicBrainz)
   |  ├── alphaTab + sintetizador e tablatura
   |  └── getUserMedia + AudioWorklet + monitor de notas
   |
 [FastAPI: API REST + tarefas]
   |  ├── ingestão/validação de áudio autorizado
   |  ├── processamento (FFmpeg/librosa)
   |  ├── Basic Pitch (audio -> MIDI/eventos)
   |  ├── Demucs (opcional para separação)
   |  ├── estimativa de acordes (P1)
   |  ├── algoritmo corda/casa
   |  ├── validação + síntese + comparação
   |  └── SQLite/arquivos temporários com limpeza
```

Use uma interface `AudioAnalysisProvider`, `SourceMetadataProvider`, `TabRenderer` e `PracticePitchDetector`, permitindo trocar bibliotecas sem reescrever a aplicação. O backend deve ser executável em CPU; modelos pesados devem ter opção de desativação e rodar em tarefa com barra de progresso **real**.

## 5. Contratos e dados

### Entidades sugeridas

- `Track`: id, title, artist, externalUrls[], userAudioId?, rightsStatus, durationMs?, provenance.
- `AnalysisJob`: id, trackId, stage, progress (0..1), status, error?, warnings[].
- `NoteEvent`: id, startMs, durationMs, midiPitch (inteiro), confidence (0..1), velocity?, sourceSegmentId.
- `ChordEvent`: startMs, durationMs, chordSymbol, confidence, evidence.
- `TabNote`: noteEventId, stringNumber (1–6, 1 = Mi aguda), fret (0..24), technique?, finger?.
- `TabVersion`: id, tuningMidiLowToHigh = [40,45,50,55,59,64], notes[], fingeringCost, pitchFidelity, timingFidelity, status ('validated'|'review_required'|'rejected'), validationDetails, previewAudioUrl?, createdAt.
- `PracticeEvent`: expectedNoteId, observedMidiPitch?, centsDeviation?, onsetDifferenceMs?, status ('correct'|'wrong_note'|'early'|'late'|'missed'|'unreliable'), confidence.

**Evitar** introduzir uma nota numa oitava diferente só porque a classe de altura coincide. Atribuição para corda/casa deve conservar o **MIDI pitch absoluto**, salvo se o usuário escolher explicitamente 'adaptação transposta'.

### Endpoints propostos

- `POST /api/sources/resolve`: metadados de URL, sem baixar streams não autorizados.
- `POST /api/audio`: upload autorizado com controle de tamanho/duração.
- `POST /api/analysis`: inicia análise; retorno `jobId` e não uma resposta demorada síncrona.
- `GET /api/analysis/{jobId}`: estágio, progresso, warnings e resultado ao concluir.
- `GET /api/tracks/{id}/tabs`: versões, avaliações, status e prévia.
- `POST /api/tabs/{id}/validate`: reexecuta verificação depois de edição.
- `POST /api/practice/session`: parâmetros para sessão; captura e processamento de mic preferencialmente no cliente.

## 6. Cadeia de validação OBRIGATÓRIA

### 6.1. Validação estrutural (bloqueante)
- Cada nota MIDI é alcançável na afinação selecionada, com casa 0..24 (configurável por instrumento).
- Uma corda não toca duas alturas simultaneamente; evitar colisão de dedos/posições.
- Acordes simultâneos exigem execução possível dentro das seis cordas, considerando pestana quando modelada; se o estimador não garantir, marcar revisão.
- Intervalos, duração, tempo e oitava não podem ser alterados silenciosamente.
- Rejeitar posições com salto de mão inviável para o tempo disponível; comparar rotas, não só acordes isolados.
- Restringir extensão de dedos, carga de movimento por segundo, cordas saltadas e técnicas avançadas conforme nível.

### 6.2. Validação musical objetiva (bloqueante quando alvo confiável)
- Converter `TabNote` de volta para notas MIDI usando afinação + número da casa.
- Comparar eventos sintetizados/reconstruídos com `NoteEvent` do alvo: altura absoluta, início, duração e simultaneidade.
- Renderizar **áudio prévio real** da solução escolhida (SoundFont de guitarra licenciado ou sintetizador compatível); disponibilizar reprodução.
- Comparar áudio sintetizado com alvo usando alinhamento temporal (p.ex. DTW), cromagrama, ataques e opcionalmente similaridade espectral. **Não** comparar timbre cru com exigência de igualdade; pedais/distorção fazem instrumentos soarem diferentes.
- Pontuar `structuralPass`, `fidelityScore`, `rhythmScore`, `ergonomicsScore`, `evidenceQuality` e agregar estado `validated|review_required|rejected`.
- A faixa `validated` refere-se **somente** aos critérios testados. Não declarar 'bonita' nem 'idêntica ao original' automaticamente.
- Usar limites calibrados com um conjunto de testes reais; não inventar um limite mágico de 95% para tudo. Registrar parâmetros no relatório.

### 6.3. Política de liberação
- Se falhar estrutura: **rejeitar e gerar alternativa**.
- Se a fonte é ruidosa/ambígua ou há desacordo forte entre áudio e renderização: **revisão necessária**, nunca 'verificada'.
- Se passar: exibir tablatura e selo textual `Validada tecnicamente` + a prévia auditiva + indicador de confiança. A aprovação musical subjetiva fica com o usuário.
- **Após qualquer alteração** de notas/posição/afinação, revalidar e atualizar status.

## 7. Otimizador de dedilhado

- Construir grafo de candidatos (`pitch` -> pares `stringNumber, fret`) e escolher caminhos por programação dinâmica (Viterbi ou shortest path).
- Função de custo: deslocamento entre posições, casas altas para iniciantes, saltos de corda, mudanças repentinas, necessidade de pestana, posições abertas e repetição de cordas quando melhora fluidez.
- Exibir opções `Fiel`, `Fácil`, `Alternativa`; uma mudança de oitava ou nota somente com consentimento explícito e indicação `Arranjo adaptado`.
- Mostrar sugestão de dedos, mas evitar afirmar que é a única digitação correta.

## 8. Prática em tempo real

- Solicitar permissão de microfone após gesto do usuário; comunicar que sem permissão a prática automática não funciona.
- Implementar `getUserMedia` + Web Audio e preferencialmente `AudioWorklet` para captura de baixa latência em HTTPS/localhost.
- Calibração de silêncio, sensibilidade, latência e afinação A4; recomendar fones de ouvido para evitar que o microfone escute a própria reprodução do aplicativo.
- P0: **uma nota por vez** com detector YIN/autocorrelação, estabilização temporal e filtro de ruído. Reconhecimento de acordes simultâneos é P2.
- Mapear frequência em MIDI + desvio em cents; corresponder ao evento esperado com janela temporal configurável e compensação da latência medida.
- Feedback note-by-note: verde correto, amarelo atrasado/adiantado/afinação marginal, vermelho incorreto, cinza 'não detectável'; a interface deve ser acessível sem depender apenas de cor.
- Não pontuar silêncio/ruído como nota errada; erros de confiança baixa devem ser `unreliable`, não penalizar indevidamente.
- Pontuar separadamente precisão melódica e temporal, não misturar em um único número opaco.
- Salvar apenas estatísticas da sessão por padrão; não gravar microfone nem transmitir áudio contínuo ao servidor sem autorização explícita.

## 9. Cifras, letra e fontes externas

- Reconhecer acordes por análise harmônica da gravação enviada (ex.: cromagrama e tempo); não presumir que é fácil identificar tríades estendidas/inversões.
- Pesquisa na web: metadados via APIs oficiais e fontes externas com termos que permitam consulta; catalogar URL, autor, data de obtenção e licença.
- Sempre distinguir: estimativa da IA, transcrição humana licenciada, revisão do usuário.
- O alinhamento de letra exige texto com direitos de exibição. Não extrair letras completas de sites não autorizados, nem replicá-las sem licença. É aceitável apresentar cifra sem letra.

## 10. Usabilidade e design

- Estética minimalista de estúdio musical (sem excessos), responsiva, alto contraste e interface totalmente em pt-BR.
- Páginas: Início/Busca, Processamento, Música/Tablatura, Praticar, Minha Biblioteca e Configurações.
- Editor de tablatura visual, linha de execução, metrônomo, trechos repetíveis, velocidade, notas destacadas e histórico de acertos.
- Mostrar limitações honestamente: `Apenas metadados disponíveis para este link`; `Transcrição parcial`; `Baixa confiança`; `Revisão necessária`.

## 11. Segurança e privacidade

- Processar upload temporário, validar conteúdo, limitar tamanho/duração, proteger contra path traversal, XXE, SSRF e injeções em processamento.
- Não deixar usuário fornecer URLs arbitrárias para downloads internos; validar origens e proibir SSRF.
- Chaves somente no backend/.env, nunca no código cliente.
- Definir política de exclusão de arquivos e armazenamento mínimo.
- Configurar CORS explicitamente, rate limits e testes de payloads indevidos.
- Testar bibliotecas e licenças: alphaTab MPL 2.0 conforme versão consultada; Basic Pitch Apache 2.0; verificar demais dependências e modelos antes do uso.

## 12. Qualidade/indicadores

Medir separadamente: (a) precisão de pitch/onset em áudio com referência real; (b) taxa de tablaturas estruturalmente executáveis; (c) latência do treino; (d) falso positivo e falso negativo do microfone; (e) desempenho por CPU; (f) transparência da taxa de erro/estado de validação. Nunca prometer acurácia geral sem benchmark externo.
