---
title: アイコンを1つのフォントにまとめたい
tags:
  - font
  - fontforge
private: false
updated_at: "2023-07-21T21:24:56+09:00"
id: cd1cbe607bb751f9bbc1
organization_url_name: qiita-inc
slide: false
---

この記事は [お題は不問！Qiita Engineer Festa 2023で記事投稿！ - Qiita](https://qiita.com/official-events/4f3daca63fb78f16df0b) の参加記事です。

https://qiita.com/official-events/4f3daca63fb78f16df0b

## はじめに

最近、[とあるフォント](https://zutomayo.net/font/)に含まれている文字を使いたいが、ひらがなにマップされているため普段遣いができず、普段遣いができるフォントの私用領域に入れて使えないかと思い、挑戦してみました。

## やったこと

グリフが空ではない部分だけを普段遣いするフォントにコピーするようにしました。
実際のスクリプトは以下です。

```python:patch.py
#!/usr/bin/env python3

import fontforge
import psMat

PROJECT_NAME = "ZUTOMOJI_HG"
PROJECT_VERSION = "0.1.0"


class Patcher:

    def __init__(self,
                 base_font_path: str,
                 ztmy_font_path: str,
                 font_name: str,
                 font_family_name: str,
                 base_start_code_point: int = 0x100000) -> None:
        self.__base_font = fontforge.open(base_font_path)
        self.__ztmy_font = fontforge.open(ztmy_font_path)
        self.__font_name = font_name
        self.__font_family_name = font_family_name
        self.__base_current_code_point = base_start_code_point

        assert self.__ztmy_font.is_cid, "{} is not CID font.".format(
            ztmy_font_path)
        subfont_count = self.__ztmy_font.cidsubfontcnt
        assert subfont_count == 1, "{} has {} subfonts. Expect only one.".format(
            ztmy_font_path, subfont_count)

        self.__em_scale: float = 0 if self.__base_font.em == self.__ztmy_font.em else (
            self.__base_font.em / self.__ztmy_font.em)

    def __patch_set(self):
        return [
            {
                "code_point_start": 0x3001,
                "code_point_end": 0x3002
            },
            {
                "code_point_start": 0xff1f,
                "code_point_end": 0xff1f
            },
            {
                "code_point_start": 0xff01,
                "code_point_end": 0xff01
            },
            {
                "code_point_start": 0x30fc,
                "code_point_end": 0x30fc
            },
            {
                "code_point_start": 0x300c,
                "code_point_end": 0x300d
            },
            {
                "code_point_start": 0x3041,
                "code_point_end": 0x308d
            },
            {
                "code_point_start": 0x308f,
                "code_point_end": 0x308f
            },
            {
                "code_point_start": 0x3092,
                "code_point_end": 0x3093
            },
            {
                "code_point_start": 0x30a1,
                "code_point_end": 0x30ed
            },
            {
                "code_point_start": 0x30ef,
                "code_point_end": 0x30ef
            },
            {
                "code_point_start": 0x30f2,
                "code_point_end": 0x30f4
            },
            {
                "code_point_start": 0x5f37,
                "code_point_end": 0x5f37
            },
            {
                "code_point_start": 0x6817,
                "code_point_end": 0x6817
            },
            {
                "code_point_start": 0x5263,
                "code_point_end": 0x5263
            },
            {
                "code_point_start": 0x9283,
                "code_point_end": 0x9283
            },
            {
                "code_point_start": 0x6ce5,
                "code_point_end": 0x6ce5
            },
            {
                "code_point_start": 0x571f,
                "code_point_end": 0x571f
            },
            {
                "code_point_start": 0x97ee,
                "code_point_end": 0x97ee
            },
            {
                "code_point_start": 0x732b,
                "code_point_end": 0x732b
            },
            {
                "code_point_start": 0x98ef,
                "code_point_end": 0x98ef
            },
            {
                "code_point_start": 0x591c,
                "code_point_end": 0x591c
            },
            {
                "code_point_start": 0x8e0a,
                "code_point_end": 0x8e0a
            },
        ]

    def __copy_glyphs(self, code_point_start: int,
                      code_point_end: int) -> None:
        self.__ztmy_font.selection.select(("ranges", "unicode"),
                                          code_point_start, code_point_end)
        glyphs = [
            g for g in self.__ztmy_font.selection.byGlyphs if g.unicode >= 0
        ]

        expect_glyph_count = code_point_end + 1 - code_point_start
        assert len(glyphs) == (
            expect_glyph_count
        ), "Invalid glyphs exist. Expected {} but actual {}.".format(
            expect_glyph_count, len(glyphs))

        for glyph in glyphs:
            print("Updating glyph: {}, at: 0x{:x}".format(
                glyph.glyphname, self.__base_current_code_point))

            self.__ztmy_font.selection.select(glyph.encoding)
            self.__ztmy_font.copy()
            self.__base_font.selection.select(self.__base_current_code_point)
            self.__base_font.paste()

            if self.__em_scale != 0:
                matrix = psMat.scale(self.__em_scale)
                self.__base_font[self.__base_current_code_point].transform(
                    matrix)

            self.__base_current_code_point += 1

    def __set_meta_data(self) -> None:
        # 略

    def patch(self) -> None:
        print("em scale: {}".format(self.__em_scale))

        for patch in self.__patch_set():
            code_point_start = patch["code_point_start"]
            code_point_end = patch["code_point_end"]

            self.__copy_glyphs(code_point_start, code_point_end)

        self.__set_meta_data()

    def generate(self, out_file_path: str) -> None:
        self.__base_font.generate(out_file_path)
        print("Generated {}".format(out_file_path))

    def close(self) -> None:
        self.__base_font.close()
        self.__ztmy_font.close()


def main():
    font_data = [
        {
            "base_font_path": "HackGenConsoleNF-Regular.ttf",
            "font_name": "ZUTOMOJI_HGC-Regular",
            "font_family_name": "ZUTOMOJI_HGC"
        },
        # 略
    ]
    ztmy_font_file = "ZTMY_MOJI-R.otf"

    for data in font_data:
        patcher = Patcher(data["base_font_path"], ztmy_font_file,
                          data["font_name"], data["font_family_name"])
        patcher.patch()

        out_file_path = "{}.ttf".format(data["font_name"])
        patcher.generate(out_file_path)
        patcher.close()


if __name__ == "__main__":
    main()
```

コピー元のフォントのグリフを`selection.select()`で選択するときに詰まったのが、

- FontForgeで開いて一番上と一番下のUnicodeコードポイントをrangesで選択していたが、一部選択されていないものがあった
  - `selection.select(('ranges', 'unicode'), 0x3001, 0x8e0a)`でだめだった

で、途中CIDを選択するように変えたりしましたが、今後とあるフォント側のアップデートでCIDが変わると大変なことになりそうだったので、
最終的にはUnicodeコードポイントを細かい単位でrangesで選択するようにしました。
（CIDフォントだからなのか？なぜ細かい単位で選択すると動くようになったのかはわかっていない...）

また、上記でも出てきたフォントがアップデートされたときのことを気にして、今のCID順に私用領域に追加し、今後文字が増えた場合は随時後ろに追加していくような方針にしました。

## 課題

最終的にはシェルのプロンプトをつくりたいなと思っているのですが、それに向けて課題が2つあります。

1つ目は、フォントサイズが小さすぎると細かい線などが見えなくなってしまいます。
以下の例だとネコの目がほとんど見えないです。
（もともと細かい線で構成されているのが原因？）

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/352836/79dea2e4-393a-d658-ac38-04d07cac9660.png)

2つ目は、ターミナル上で文字が重なってしまっている点です。（少なくとも僕の環境では）
調べてみるとAmbiguous Width問題というものらしく、僕の環境（Alacritty, Zsh, Zellij）でうまく表示できないかいじってみる予定です。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/352836/4643e178-382e-bc64-1302-3cd616887311.png)

## Refs

- [FontForge 講座](https://aznote.jakou.com/fforge/index.html)
- [フォントのフォーマットに気をつけよう | 技術情報 | InDesign（インデザイン）専門の質の高いDTP制作会社―株式会社インフォルム](https://www.informe.co.jp/useful/character/character3/)
- [nerd-fonts/font-patcher at v3.0.2 · ryanoasis/nerd-fonts](https://github.com/ryanoasis/nerd-fonts/blob/v3.0.2/font-patcher)
- [Nerd fontとpowerlineとambiguous width - Qiita](https://qiita.com/s417-lama/items/b38089a42fe7d4a061da)
- [Macのターミナルで全角記号を綺麗に表示したかった - Qiita](https://qiita.com/seto-r/items/706d2c4d795dba7d42ed)
- [ambiwidth patches](https://gist.github.com/lo48576/194f05f9266761d6925495237594edbc)
- [tobijibu-ashiato: tmux + Vimで全角記号のずれをなおす](https://you84815.blogspot.com/2018/08/tmux-vim.html)
