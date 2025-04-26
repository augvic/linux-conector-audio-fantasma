# Solução para bug de "dispositivo fantasma de fones de ouvido" no Linux (Fedora)

## Problema

No Fedora 42, ao conectar fones de ouvido na saída traseira (verde), um "dispositivo fantasma" de fones de ouvido frontal aparecia e desaparecia rapidamente, causando estouros (pops) no áudio. Isso acontecia mesmo com o painel frontal fisicamente desconectado da placa-mãe.

## Diagnóstico

1. **Dispositivo de áudio listado no PulseAudio/PipeWire:**
   - A saída de som aparecia como "Starship/Matisse HD Audio Controller".
   - A porta alternava entre "Saída de linha (conectado)" e "Fones de ouvido (desconectado)".

2. **Verificação com `pavucontrol`:**
   - Identificava mudança rápida para "Fones de ouvido" no momento dos estouros.

3. **Testes com `hda-verb` e `cat /proc/asound/card0/codec#0`:**
   - Mostraram que a entrada de áudio estava intermitente, mas sem controle direto funcional via terminal.

4. **Alteração do perfil de áudio para "Pro Audio":**
   - Diminuiu significativamente os estouros, mas não eliminou completamente.

## Solução

A ferramenta `HDAJackRetask`, parte do pacote `alsa-tools`, foi utilizada para mapear os pinos de áudio corretamente e desabilitar o pino frontal que causava os estouros.

### Passos executados:

1. **Instalação da ferramenta:**
```bash
sudo dnf install alsa-tools
```

2. **Abrir o `HDAJackRetask`:**
```bash
sudo hdajackretask
```

3. **Selecionar o codec correto (geralmente "Realtek ALCXXX" ou similar).**

4. **Identificar o pino correspondente ao painel frontal de fone de ouvido.**
   - No caso observado, era o pino `0x19` (nomeado como "Fones de ouvido, painel frontal").

5. **Marcar a opção "Override" para esse pino e definir como "Not connected".**

6. **Clicar em "Install boot override" (e *não* em "Apply now").**

7. **Reiniciar o sistema.**

Após isso, o dispositivo fantasma desapareceu por completo e os estouros cessaram totalmente.

---

### Observações finais
- Nenhum arquivo adicional precisa ser deixado no sistema após o uso do `hdajackretask`, desde que se use a opção "Install boot override".
- A experiência com áudio pode melhorar ainda mais ao selecionar o perfil de áudio correto no `pavucontrol` (em alguns casos, "Pro Audio" realmente ajuda).
- A desativação do painel frontal é segura desde que o usuário não precise utilizá-lo.

---

💡 **Aprendizado:**
Esse processo é uma excelente demonstração do poder de customização do Linux e de como o conhecimento sobre hardware e drivers pode ajudar a resolver problemas aparentemente insolúveis.