## コンセプト
- すべての Bus には「オプトコンプ」が挿入させており、多段式にコンプレッションする前提
- 自然に目立たせたい音は多段式コンプレッションをパイパスしている (例：FX や WetVox)

```mermaid


stateDiagram
    %% 楽器・ボーカルの入力
    [*] --> Synth : シンセ音源入力
    [*] --> Kick : キック入力
    [*] --> Bass : ベース入力
    [*] --> Snare : スネア入力
    [*] --> HiHat : ハイハット入力
    [*] --> ImpactFX : 効果音1
    [*] --> SweepFX : 効果音2
    [*] --> Vox : ボーカル入力

    %% Busへのルーティング
    Synth --> SynthBus
    Kick --> DrumBus
    Snare --> DrumBus
    HiHat --> DrumBus
    ImpactFX --> FXBus
    SweepFX --> FXBus
    Kick --> Bass: サイドチェイン（キックをトリガーにベース抑制）
    Bass --> BassBus

    %% ボーカル処理：DryとReverb Sendに分岐
    Vox --> DryVoxBus : 原音保持
    Vox --> ReverbPlateVox : PlateリバーブへSend
    Vox --> ReverbRoomVox : RoomリバーブへSend

    %% リバーブ音をWetVoxBusへ統合
    ReverbPlateVox --> WetVoxBus
    ReverbRoomVox --> WetVoxBus

    %% PreVoxでDry/Wetを合流
    DryVoxBus --> VoxBus
    WetVoxBus --> VoxBus

    %% KaraokeBusはVoxを含まずInstrumentのみ
    SynthBus --> InstrumentBus
    DrumBus --> InstrumentBus

    %% 各バスをPreMasterへ統合
    FXBus --> KaraokeBus: InstrumentBus上の多段コンプをバイパスして自然な出力
        BassBus --> KaraokeBus: InstrumentBus上の多段コンプをバイパスして自然な出力
    InstrumentBus --> KaraokeBus
    KaraokeBus --> PreMaster
    VoxBus --> PreMaster

    %% Master出力
    PreMaster --> Master

    Master --> [*]
```

