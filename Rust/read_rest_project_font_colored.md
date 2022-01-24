---
title: read_rest_project_font_colored
date: 2021-04-30 10:58:32
categories:  [rust]
tags: [rust]
---


<!--more-->


# read_rest_project_font_colored


```rust

let xx = "xx".red()
let xx = "xx".green()

pub trait Colorize {
  fn red(self) -> super::ColoredString;
  fn black(self) -> super::ColoredString;
  fn green(self) -> super::ColoredString;
}

pub struct ColoredString {
    input: String,
    fgcolor: Option<Color>,
    bgcolor: Option<Color>,
    style: Style,
}

impl<'a> Colorize for &'a str {

    fn red(self) -> ColoredString {
        ColoredString {
            input: String::from(self),
            fgcolor: Some(Color::Red),
            .. ColoredString::default()
        }
    }

    def_str_color!(fgcolor: green => Color::Green);
    

    macro_rules! def_str_color {
        ($side:ident: $name: ident => $color: path) => {
            fn $name(self) -> ColoredString {
                ColoredString {
                    input: String::from(self),
                    $side: Some($color),
                    .. ColoredString::default()
                }
            }
        }
    }

    // After the transformation , marco like this
    
    fn green() -> ColoredString{
        ColoredString{
            input: String::from(self),
            fgcolor: Some(Color::Red),
            .. ColoredString::default()
        }
    }

}

```