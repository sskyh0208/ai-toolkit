# ai-toolkit

AI コーディングエージェント（Claude Code / Cursor / GitHub Copilot など）に渡す、自作のスキルなどを集めたパッケージです。[apm](https://github.com/microsoft/apm)（Agent Package Manager）で各リポジトリへ配ります。

自分用に少しずつ育てています。何が入っているかは `.apm/skills/` 配下の各 `SKILL.md` を見てください。

## 使い方

apm CLI を入れます。

```bash
curl -sSL https://aka.ms/apm-unix | sh
```

使いたいプロジェクトの `apm.yml` に依存を足します。

```yaml
dependencies:
  apm:
    - <org>/ai-toolkit#v2026.07.01.1   # リリースタグで固定
```

`apm install` で各ツールの読み込み先（`.claude/` `.cursor/` `.agents/skills/` など）へ展開されます。

## 構成

```
.
├── apm.yml                     # このパッケージのマニフェスト
├── apm.yml.example             # 取り込む側に置く apm.yml の例
├── .apm/
│   └── skills/                 # スキル本体
└── .github/workflows/
    └── release.yml             # main マージで CalVer タグとリリースを作成
```

## リリース

`main` にマージされると `.github/workflows/release.yml` が CalVer タグ `vYYYY.MM.DD.N` を切り、リリースノートを自動生成します。取り込む側はこのタグで固定します。

## ライセンス

このリポジトリは MIT です（`LICENSE` 参照）。
