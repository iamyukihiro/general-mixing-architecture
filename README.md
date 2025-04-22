# 汎用的なミキシング・アーキテクチャ

## コンセプト
- 圧縮した音源に対して、原音やセンドリバーブをマージすることでトランジェントの復活、自然な空気感の演出を行う目的があります。
- すべての Bus には「オプトコンプ」が挿入させており、多段式にコンプレッションすることで、自然なピークリダクションを狙っています。
- 自然に目立たせたい音は多段式コンプレッションをバイパスしています (例：FX や WetVox)
- InstrumentBus、もしくはそこに集まるBusの音の特性上、アタックが強いがトランジェントが少なく、ブリックウォール・リミッターを用いてピークを下げてください。
  - 但し、聴覚上、極端な音の変化が変わる場合は行わないでください。あくまでピークを下げることを目的としてください。

## ダイアグラム

```mermaid
stateDiagram
    %% 楽器・ボーカルの入力
    [*] --> ■Synth : シンセ音源入力
    [*] --> ■Kick : キック入力
    [*] --> ■Bass : ベース入力
    [*] --> ■Snare : スネア入力
    [*] --> ■HiHat : ハイハット入力
    [*] --> ■ImpactFX : 効果音1
    [*] --> ■SweepFX : 効果音2
    [*] --> ■Vox : ボーカル入力

    %% Busへのルーティング
    ■Synth --> SynthBus
    ■Kick --> DrumBus
    ■Snare --> DrumBus
    ■HiHat --> DrumBus
    ■ImpactFX --> FXBus
    ■SweepFX --> FXBus
    ■Kick --> ■Bass : Kickによるサイドチェイン制御
    ■Bass --> BassBus

    %% ボーカル処理：DryとReverb Sendに分岐
    ■Vox --> DryVoxBus : 原音保持
    ■Vox --> ReverbPlateVox : PlateリバーブへSend
    ■Vox --> ReverbRoomVox : RoomリバーブへSend

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
    FXBus --> NaturalBus: 
    note right of NaturalBus : InstrumentBus上の多段コンプをバイパスして自然な出力を行う
    BassBus --> NaturalBus
    NaturalBus --> KaraokeBus : 自然な出力音
    KaraokeBus --> PreMaster
    note right of KaraokeBus : マスタリング EQ を用いてオプトコンプで付与された中域の倍音成分を整える
    note right of KaraokeBus : マスタリング EQ を用いて Vox 用の帯域を削る
               
    InstrumentBus --> KaraokeBus : 多段圧縮された音
    VoxBus --> PreMaster
     note left of VoxBus : マスタリング EQ や ダイナミック EQ、リニアフェイズEQ を用いて不要な帯域を削る

    %% Master出力
    PreMaster --> Master
    note right of PreMaster : マスタリング EQ を用いて最終的な全体のトーンを整える
    note right of PreMaster : 2mix

    Master --> [*]
    note right of Master : 簡易のマスタリング
```

