# kaeruglobal/chrono — 非公式の Fork（kaeru 版）

> **これは [chronotope/chrono](https://github.com/chronotope/chrono) の非公式な Fork です。crates.io には公開しません。**
> kaeruglobal の内部（RanaDeviceManager）が、`[patch.crates-io]` から固定の SHA で参照するためだけに置いています。
> ライセンスは上流と同じ **MIT OR Apache-2.0**（`LICENSE.txt` を参照。変更していません）。
> 上流の README は、この節の下にそのまま残しています。

## なぜ Fork したか
- 2026-09-14 に RUSTSEC-2026-0543 で chrono が「保守されていない（unmaintained）」と登録された（修正版なし。chrono#1768 でメンテナーが活動を縮小）。
- RanaDeviceManager は chrono を **自分では使っておらず**、`openidconnect → oauth2 → chrono`（と `openidconnect → chrono`）の推移依存として持つ。
  上流（openidconnect-rs #256、oauth2-rs #351・PR #355）はまだ移行しておらず、時期の見通しも立たない。
- そのため、代替が出るまでの間、**未リリースの直しを取り込んだ Fork を使い、CI の audit/deny を ignore なしで通す**。

## この Fork の中身（`kaeru-0.4` 枝 = 既定の枝）
`main` は上流の `main` をそのまま追う枝（触らない）。`kaeru-0.4` は、上流の **v0.4.45**（`1703382`）に、次の直しだけを載せたもの。

上流で取り込み済み（未リリース）の直し 6 件（`git cherry-pick -x`）:
| 上流のコミット | 内容 | 取り込む理由 |
|---|---|---|
| `75926fce5` | `find_local_time_type_from_local` の i64 オーバーフロー回避 | 時刻計算の panic / 誤り（安全） |
| `a71c212cd` | 部分的な tzdata の索引で範囲外のスライスを避ける | 壊れた入力で panic（安全） |
| `3ffcd1b10` | オフセットの分の丸めで不正な `+24:00` を出さない | 不正な値の生成（正しさ） |
| `2c8426a3b` | 長い時間帯の略称・`_` を TZif で受け付ける | `/etc/localtime` が読めず UTC に黙って落ちる不具合 |
| `7b6436f65` | `NaiveDate` の日/週イテレーターの逆順の誤り | 範囲外の日付を返す誤り |
| `6adaa5240` | `from_isoywd_opt` が範囲外の年で panic しない | `_opt` なのに panic |

上流で未マージの PR 1 件:
- `chronotope/chrono#1812`（TZif のヘッダーの長さ計算を `checked_mul` に。壊れた入力で 32 ビット環境の panic を防ぐ）。変更が小さく、テスト付き、挙動の変化は「panic → エラー」だけ。

**取り込まなかったもの**: CI・依存の更新だけのコミット（checkout/setup-node の更新、clippy の指摘、wasm の cfg、CI の予定実行）。
挙動を変える未マージ PR（#1811 `From<SystemTime>` の飽和、#1801 オフセット秒の解析、#1813 高速な整形、他の機能追加）は、上流の判断が出ていないので入れていない。
バージョンは上流と同じ `0.4.45` のまま（`+` などで変えると、依存側の `^0.4` の指定との照合や勧告データベースの照合の挙動が変わるため）。見分けは**タグと SHA** で行う。

## タグ
`v0.4.45-kaeru.N`（N は 1 から）。参照は**タグではなく固定の SHA**（`rev = "…"`）で行う。タグは目印。

## 上流に戻す条件（この Fork をやめる）
次のどれかが満たされたら、`[patch.crates-io]` を外して crates.io の版に戻す（Fork は残してアーカイブ）:
1. oauth2 と openidconnect の**両方**が chrono を使わない版（jiff / time へ移行）をリリースした。そして RanaDeviceManager がそれに上げられた。
2. chrono の勧告（RUSTSEC-2026-0543）が取り下げられた、または chrono に新しいメンテナー・修正版が出た。
3. 上流が上の直しを含む版をリリースし、かつ 1・2 のどちらかが成り立つ。

## 運用の約束
- **Fork に載せる変更は、上の一覧の取り込みだけ**。新しい機能・独自の変更は足さない。
- 上流の新しい安全の直しは、毎週の見回し（RanaDeviceManager の `reports/ranadm-chrono-alternative/WATCH.md`）で見て、選んで取り込み、新しいタグ（`-kaeru.N+1`）と新しい SHA にする。
- この Fork では GitHub Actions を有効にしていない（Fork 先の CI を回さない）。試験は手元（コンテナ）で `cargo test` を流して確かめる。
- 書き込み権限は kaeruglobal の管理者のみ。

---

[Chrono][docsrs]: Timezone-aware date and time handling
========================================

[![Chrono GitHub Actions][gh-image]][gh-checks]
[![Chrono on crates.io][cratesio-image]][cratesio]
[![Chrono on docs.rs][docsrs-image]][docsrs]
[![Chat][discord-image]][discord]
[![codecov.io][codecov-img]][codecov-link]

[gh-image]: https://github.com/chronotope/chrono/actions/workflows/test.yml/badge.svg?branch=main
[gh-checks]: https://github.com/chronotope/chrono/actions/workflows/test.yml?query=branch%3Amain
[cratesio-image]: https://img.shields.io/crates/v/chrono.svg
[cratesio]: https://crates.io/crates/chrono
[docsrs-image]: https://docs.rs/chrono/badge.svg
[docsrs]: https://docs.rs/chrono
[discord-image]: https://img.shields.io/discord/976380008299917365?logo=discord
[discord]: https://discord.gg/sXpav4PS7M
[codecov-img]: https://img.shields.io/codecov/c/github/chronotope/chrono?logo=codecov
[codecov-link]: https://codecov.io/gh/chronotope/chrono

Chrono aims to provide all functionality needed to do correct operations on dates and times in the
[proleptic Gregorian calendar](https://en.wikipedia.org/wiki/Proleptic_Gregorian_calendar):

* The [`DateTime`](https://docs.rs/chrono/latest/chrono/struct.DateTime.html) type is timezone-aware
  by default, with separate timezone-naive types.
* Operations that may produce an invalid or ambiguous date and time return `Option` or
  [`MappedLocalTime`](https://docs.rs/chrono/latest/chrono/offset/enum.MappedLocalTime.html).
* Configurable parsing and formatting with an `strftime` inspired date and time formatting syntax.
* The [`Local`](https://docs.rs/chrono/latest/chrono/offset/struct.Local.html) timezone works with
  the current timezone of the OS.
* Types and operations are implemented to be reasonably efficient.

Timezone data is not shipped with chrono by default to limit binary sizes. Use the companion crate
[Chrono-TZ](https://crates.io/crates/chrono-tz) or [`tzfile`](https://crates.io/crates/tzfile) for
full timezone support.

## Documentation

See [docs.rs](https://docs.rs/chrono/latest/chrono/) for the API reference.

## Limitations

* Only the proleptic Gregorian calendar (i.e. extended to support older dates) is supported.
* Date types are limited to about +/- 262,000 years from the common epoch.
* Time types are limited to nanosecond accuracy.
* Leap seconds can be represented, but Chrono does not fully support them.
  See [Leap Second Handling](https://docs.rs/chrono/latest/chrono/naive/struct.NaiveTime.html#leap-second-handling).

## Crate features

Default features:

* `alloc`: Enable features that depend on allocation (primarily string formatting).
* `std`: Enables functionality that depends on the standard library. This is a superset of `alloc`
  and adds interoperation with standard library types and traits.
* `clock`: Enables reading the local timezone (`Local`). This is a superset of `now`.
* `now`: Enables reading the system time (`now`).
* `wasmbind`: Interface with the JS Date API for the `wasm32` target.

Optional features:

* `serde`: Enable serialization/deserialization via [serde].
* `rkyv`: Deprecated, use the `rkyv-*` features.
* `rkyv-16`: Enable serialization/deserialization via [rkyv], using 16-bit integers for integral `*size` types.
* `rkyv-32`: Enable serialization/deserialization via [rkyv], using 32-bit integers for integral `*size` types.
* `rkyv-64`: Enable serialization/deserialization via [rkyv], using 64-bit integers for integral `*size` types.
* `rkyv-validation`: Enable rkyv validation support using `bytecheck`.
* `arbitrary`: Construct arbitrary instances of a type with the Arbitrary crate.
* `unstable-locales`: Enable localization. This adds various methods with a `_localized` suffix.
  The implementation and API may change or even be removed in a patch release. Feedback welcome.
* `oldtime`: This feature no longer has any effect; it used to offer compatibility with the `time` 0.1 crate.

Note: The `rkyv{,-16,-32,-64}` features are mutually exclusive.

[serde]: https://github.com/serde-rs/serde
[rkyv]: https://github.com/rkyv/rkyv

## Rust version requirements

The Minimum Supported Rust Version (MSRV) is currently **Rust 1.62.0**.

The MSRV is explicitly tested in CI. It may be bumped in minor releases, but this is not done
lightly.

## License

This project is licensed under either of

* [Apache License, Version 2.0](https://www.apache.org/licenses/LICENSE-2.0)
* [MIT License](https://opensource.org/licenses/MIT)

at your option.
