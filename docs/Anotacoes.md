# Anotacoes do grupo

## Onde o programa começa

O ponto de entrada é  o arquivo `src/main.c`.

A funcao `main` faz três coisas em sequência:
1. Chama `parse_args()` para interpretar os argumentos da linha de comando e preencher `struct trace_options`.
2. Inicializa `struct trace_state`, que carrega o flag `--raw-events` e o pareador de syscalls.
3. Chama `trace_program()` passando o argv do alvo, a função callback `trace_observer` e o estado.


## Onde o processo alvo é criado

O processo alvo é criado em `src/trace_runtime.c`

O fluxo da funcao é:
- O pai chama `fork()`.
- O filho executa, em ordem:
  - `ptrace(PTRACE_TRACEME, 0, NULL, NULL)`, avisa ao kernel que será rastreado pelo pai.
  - `raise(SIGSTOP)`, para voluntariamente, para que o pai consiga sincronizar com o filho antes dele executar qualquer coisa.
  - `execvp(argv[0], argv)`, substitui a imagem do processo pelo programa alvo.
- O **pai** apenas retorna o PID do filho.

A função `wait_for_initial_stop()` é quem faz o pai esperar o `SIGSTOP` do filho antes de continuar.

---

## Onde o runtime chama o callback

Em `src/trace_runtime.c`, dentro de `trace_program()`, no loop principal.

Depois de cada parada detectada como syscall-stop, o runtime:
1. Lê os registradores com `PTRACE_GETREGS`.
2. Chama `fill_event_from_regs()` para montar o `struct syscall_event`.
3. Chama `observer(&ev, userdata)`, que é o callback registrado em `main.c`.

O callback em `main.c` é `trace_observer()`. Decidindo:
- Se `--raw-events` estiver ativo: chama `student_debug_raw_event()` e imprime.
- Caso contrário: chama `student_pair_syscall()` para combinar entrada e saída; quando o par estiver completo, chama `student_format_event()` e imprime.

---

## Quais arquivos o grupo deve modificar
Os arquivos que devem ser modificados sao:
- trace_runtime.c
- pairer.c
- formatter.c

Os arquivos em `include/` e `src/cli.c` não precisam ser modificados.

---

## Qual TODO aparece primeiro ao executar o scaffold


O primeiro TODO ao executar é:

```
erro: TODO Semana 2: implementar launch_tracee()
```

Ela vem de `src/trace_runtime.c`, dentro de `launch_tracee()`. O que mostra que essa funcao deve ser o ponto de partida do projeto.

