from kivy.app import App
from kivy.core.window import Window
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.gridlayout import GridLayout
from kivy.uix.label import Label
from kivy.uix.button import Button
from kivy.graphics import Color, Ellipse

ANSWER = "пошел нахуй сам считай"
Window.clearcolor = (0, 0, 0, 1)

DARK = (0.10, 0.10, 0.10, 1)
GREY = (0.16, 0.16, 0.16, 1)
TEAL = (0.10, 0.48, 0.42, 1)
RED = (0.90, 0.40, 0.40, 1)
WHITE = (1, 1, 1, 1)


class CircleButton(Button):
    def __init__(self, bg=DARK, fg=WHITE, **kwargs):
        super().__init__(**kwargs)
        self.background_normal = ""
        self.background_down = ""
        self.background_color = (0, 0, 0, 0)
        self.color = fg
        self.font_size = 42
        with self.canvas.before:
            Color(*bg)
            self._circle = Ellipse(pos=self.pos, size=self.size)
        self.bind(pos=self._update, size=self._update)

    def _update(self, *args):
        self._circle.pos = self.pos
        self._circle.size = self.size


class Calc(BoxLayout):
    def __init__(self, **kwargs):
        super().__init__(orientation="vertical", padding=20, spacing=10, **kwargs)
        self.expr = ""
        self.showing_answer = False

        # верхняя панель с иконками
        top = BoxLayout(size_hint_y=None, height=90)
        top.add_widget(Label(text="↺", font_size=50, color=(0.7, 0.7, 0.7, 1),
                             size_hint_x=0.2, halign="left"))
        top.add_widget(Label(size_hint_x=0.6))
        top.add_widget(Label(text="√", font_size=50, color=(0.7, 0.7, 0.7, 1),
                             size_hint_x=0.2, halign="right"))
        self.add_widget(top)

        # дисплей
        self.display = Label(text="0", font_size=110, halign="right", valign="bottom",
                             size_hint_y=0.45)
        self.display.bind(size=lambda lbl, s: setattr(lbl, "text_size", s))
        self.add_widget(self.display)

        # сетка кнопок
        grid = GridLayout(cols=4, spacing=18, size_hint_y=0.55)
        keys = [
            ("C", GREY, RED), ("⌫", GREY, RED), ("%", GREY, WHITE), ("÷", GREY, WHITE),
            ("7", DARK, WHITE), ("8", DARK, WHITE), ("9", DARK, WHITE), ("×", GREY, WHITE),
            ("4", DARK, WHITE), ("5", DARK, WHITE), ("6", DARK, WHITE), ("-", GREY, WHITE),
            ("1", DARK, WHITE), ("2", DARK, WHITE), ("3", DARK, WHITE), ("+", GREY, WHITE),
            ("( )", DARK, WHITE), ("0", DARK, WHITE), (",", DARK, WHITE), ("=", TEAL, WHITE),
        ]
        for text, bg, fg in keys:
            btn = CircleButton(text=text, bg=bg, fg=fg)
            btn.bind(on_press=self.on_key)
            grid.add_widget(btn)
        self.add_widget(grid)

    def on_key(self, btn):
        key = btn.text

        if key == "C":
            self.expr = ""
            self.showing_answer = False
            self.display.text = "0"
        elif key == "⌫":
            if not self.showing_answer:
                self.expr = self.expr[:-1]
                self.display.text = self.expr or "0"
        elif key == "=":
            self.showing_answer = True
            self.display.text = ANSWER
        else:
            # после ответа новый ввод начинается с нуля
            if self.showing_answer:
                self.expr = ""
                self.showing_answer = False
            self.expr += key
            self.display.text = self.expr


class CalcApp(App):
    def build(self):
        return Calc()


if __name__ == "__main__":
    CalcApp().run()
