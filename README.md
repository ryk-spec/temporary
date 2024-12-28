## privateも含め，書いたコード

- `pyplot_molsci`:研究で使うフォーマットの`matplotlib`コード（`fig`,`ax`オブジェクトを改造）
- `gmx_GUI`:`gromacs`を`flet`から使うのが目標。今はバックエンド部を作成中。

```python

import flet as ft

class DynamicParameterCollector(ft.UserControl):
    def __init__(self, parameter_config):
        """
        :param parameter_config: パラメータ設定 (key: パラメータ名, value: 入力形式情報)
            例:
            {
                "text_param": {"type": "text", "label": "テキスト入力", "default": ""},
                "dropdown_param": {"type": "dropdown", "label": "選択", "options": ["Option1", "Option2"]},
                "file_param": {"type": "file", "label": "ファイル選択"}
            }
        """
        super().__init__()
        self.parameter_config = parameter_config
        self.inputs = {}  # 各ウィジェットを格納する辞書

    def get_parameters(self):
        """
        各入力ウィジェットから値を取得し、辞書形式で返す。
        """
        result = {}
        for name, widget in self.inputs.items():
            if isinstance(widget, ft.TextField):
                result[name] = widget.value
            elif isinstance(widget, ft.Dropdown):
                result[name] = widget.value
            elif isinstance(widget, ft.FilePickerResultEvent):
                result[name] = widget.files[0].name if widget.files else None
        return result

    def build(self):
        """
        ウィジェットを動的に生成。
        """
        widgets = []
        for name, config in self.parameter_config.items():
            label = config.get("label", name)
            if config["type"] == "text":
                field = ft.TextField(label=label, value=config.get("default", ""), width=300)
                self.inputs[name] = field
                widgets.append(ft.Row([field]))
            elif config["type"] == "dropdown":
                field = ft.Dropdown(
                    label=label,
                    options=[ft.dropdown.Option(opt) for opt in config.get("options", [])],
                )
                self.inputs[name] = field
                widgets.append(ft.Row([field]))
            elif config["type"] == "file":
                file_picker = ft.FilePicker()
                file_picker_button = ft.ElevatedButton(
                    text=label,
                    on_click=lambda e, p=file_picker: p.pick_files()
                )
                self.inputs[name] = file_picker
                widgets.append(ft.Row([file_picker_button]))

        return ft.Column(widgets, spacing=10)

# メインアプリケーション
def main(page: ft.Page):
    page.title = "動的パラメータ入力"
    page.scroll = "auto"

    # パラメータ設定
    parameter_config = {
        "text_param": {"type": "text", "label": "テキスト入力", "default": "デフォルト値"},
        "dropdown_param": {"type": "dropdown", "label": "ドロップダウン選択", "options": ["Option1", "Option2", "Option3"]},
        "file_param": {"type": "file", "label": "ファイル選択"}
    }

    # パラメータ収集UI
    collector = DynamicParameterCollector(parameter_config)

    # 実行ボタンのクリックイベント
    def on_execute(e):
        parameters = collector.get_parameters()
        page.add(ft.Text(f"収集されたパラメータ: {parameters}"))

    execute_button = ft.ElevatedButton("パラメータを取得", on_click=on_execute)

    # UIの追加
    page.add(collector, execute_button)

# アプリケーションの実行
ft.app(target=main)
```