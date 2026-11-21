# Super Sabicom

Biblioteca portátil de emulação de **SNES / Super Famicom** em Rust. É o núcleo usado pelo crate `emmu-snes` no caminho WebAssembly do Emmu.

> Projeto upstream: [tanakh/super-sabicom](https://github.com/tanakh/super-sabicom), pacote `super-sabicom` 0.2.0. Licença **MIT**, `Copyright (c) 2022 Hideyuki Tanaka`.

## Uso no Emmu

- `crates/emmu-snes` depende deste crate por caminho relativo.
- O wrapper `SnesCore` adiciona reparos de cabeçalho, saída RGBA8888 256×224, áudio estéreo `i16`, controle de 2 jogadores, save/load state e bindings `wasm-bindgen`.
- O workspace raiz do Emmu **exclui** este submódulo para que os dev-dependencies `compress-tools` (exige `libarchive`) e `sdl2` não sejam compilados pelo `cargo check --workspace`.
- No desktop, SNES é executado pelo Mesen; este core é usado somente no navegador.

## Recursos implementados

- CPU 65816, PPU, SPC700 e DSP.
- `Context` orquestra CPU/PPU/SPC/barramento a cada frame.
- Implementa `meru_interface::EmulatorCore` (`system_name: "SNES (Super Sabicom)"`, extensões `sfc`, `smc`, `swc`, `fig`).
- Save state: `bincode::serialize(&ctx)` / `deserialize`, com restauração do mapeamento do cartucho.
- Backup RAM e reset preservando ROM + save RAM.
- `game_info()` expõe título, código, velocidade, map mode, chipset, tamanhos de ROM/SRAM, país e desenvolvedor.
- Quatro slots de controle; P1 mapeado por padrão para A=X, B=Z, X=S, Y=A, L=Q, R=W, Start=Return, Select=LShift e setas para o D-pad.

## Funcionalidades não implementadas

Conforme o README original do projeto:

- PPU: BG Mode 6, Direct Color Mode, Interlaced Mode.
- Chips de melhoria: Super FX, CX4, DSP, S-DD1, S-RTC e SA1.

Consequentemente, jogos que exigem esses recursos não rodam no caminho WASM do Emmu.

## Árvore de arquivos

```text
emmu-super-sabicom/
├── .cargo_vcs_info.json     # sha1 upstream e897aef...
├── .cargo-ok
├── Cargo.toml               # manifesto normalizado do pacote
├── Cargo.toml.orig          # manifesto original com super-sabicom-macro
├── LICENSE                  # MIT, Copyright (c) 2022 Hideyuki Tanaka
├── README.md
├── examples/
│   ├── 65816-test.rs        # teste de CPU com SDL2
│   ├── mini.rs              # exemplo mínimo com SDL2
│   ├── spc.rs               # exemplo de áudio SPC
│   └── spc700-test.rs       # teste SPC700
└── src/
    ├── lib.rs               # Snes, Config, Error, EmulatorCore, save/load
    ├── context.rs           # orquestração dos componentes
    ├── cpu.rs               # 65816
    ├── ppu.rs               # PPU e framebuffer
    ├── spc.rs               # SPC700
    ├── dsp.rs               # DSP de áudio
    ├── bus.rs               # barramento
    ├── cartridge.rs         # cartucho e backup RAM
    ├── rom.rs               # leitura/parse de ROM e cabeçalho
    └── consts.rs            # constantes
```

## Build e teste

Este crate não é membro do workspace do Emmu, mas pode ser compilado diretamente:

```sh
cargo build
cargo test
```

Os exemplos usam SDL2; em Linux pode ser necessário instalar os headers de desenvolvimento do SDL2 e `libarchive` para os dev-dependencies.

Para gerar o WASM usado pelo Emmu, entre em `../emmu-snes` e rode `wasm-pack build --target web ...` (o comando completo está em `platforms/web/ui/package.json`, script `build:wasm`).

## Licença

MIT. Texto integral em [`LICENSE`](LICENSE):

```text
MIT License
Copyright (c) 2022 Hideyuki Tanaka
```

Não há código de terceiros vendorizado dentro deste submódulo. As dependências `bincode`, `educe`, `encoding_rs`, `log`, `meru-interface`, `modular-bitfield`, `schemars`, `serde`, `super-sabicom-macro` e `thiserror` mantêm suas próprias licenças no crates.io.
