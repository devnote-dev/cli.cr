# CLI.cr

Yet another command line interface library for Crystal. Based on [spf13/cobra](https://github.com/spf13/cobra), CLI.cr is built to be almost entirely modular, giving you absolute control over almost everything without the need for embedded macros - there isn't even a default help command or flag!

## Installation

1. Add the dependency to your `shard.yml`:
```yaml
dependencies:
  cli:
    github: devnote-dev/cli.cr
    branch: stable
```

2. Run `shards install`

## Basic Usage

```crystal
require "cli"

class MainCommand < CLI::Command
  def setup : Nil
    @name = "greet"
    @description = "Greets a person"
    add_argument "name", description: "the name of the person to greet", required: true
    add_option 'c', "caps", description: "greet with capitals"
    add_option 'h', "help", description: "sends help information"
  end

  def pre_run(arguments : CLI::ArgumentsInput, options : CLI::OptionsInput) : Bool
    if options.has? "help"
      puts help_template # generated using CLI::Formatter

      false
    else
      true
    end
  end

  def run(arguments : CLI::ArgumentsInput, options : CLI::OptionsInput) : Nil
    message = "Hello, #{arguments.get("name")}!"

    if options.has? "caps"
      puts message.upcase
    else
      puts message
    end
  end
end

main = MainCommand.new
main.execute ARGV
```

```
$ crystal greet.cr -h
Usage:
        greet <arguments> [options]

Arguments:
        name    the name of the person to greet (required)

Options:
        -c, --caps  greet with capitals
        -h, --help  sends help information

$ crystal greet.cr Dev
Hello, Dev!

$ crystal greet.cr -c Dev
HELLO, DEV!
```

## Commands

By default, the `Command` class is initialized with almost no values. All information about the command must be defined in the `setup` method.

```crystal
class MainCommand < CLI::Command
  def setup : Nil
    # prefer using `@name =` instead of `name =` to avoid method conflicts
    @name = "greet"
    # same here
    @description = "Greets a person"
    # defines an argument
    add_argument "name", description: "the name of the person to greet", required: true
    # defines a flag option
    add_option 'c', "caps", description: "greet with capitals"
    add_option 'h', "help", description: "sends help information"
  end
end
```
> **Note**
> See [command.cr](/src/cli/command.cr) for the full list of options.

Commands can also contain children, or subcommands:
```crystal
require "cli"
# import our subcommand here
require "./welcome_command"

# using the `MainCommand` created earlier
main = MainCommand.new
main.add_command WelcomeCommand.new
# there is also the `add_commands` method for adding multiple
# subcommands at one time

# run the command
main.execute ARGV
```

```$ crystal greet.cr -h
Usage:
        greet <arguments> [options]

Commands:
        welcome    sends a friendly welcome message

Arguments:
        name    the name of person to greet (required)

Options:
        -c, --caps  greet with capitals
        -h, --help  sends help information

$ crystal greet.cr welcome Dev
Welcome to the CLI world, Dev!
```

As well as being able to have subcommands, they can also inherit certain properties from the parent command:
```crystal
# in welcome_command.cr ...
class WelcomeCommand < CLI::Command
  def setup : Nil
    # ...

    # this will inherit the header and footer properties
    inherit_borders = true
    # this will NOT inherit the parent flag options
    inherit_options = false
    # this will inherit the input, output and error IO streams
    inherit_streams = true
  end
end
```

## Arguments and Options

Arguments and flag options can be defined in the `setup` method of a command using the `add_argument` and `add_option` methods respectively.
```crystal
class MainCommand < CLI::Command
  def setup : Nil
    add_argument "name",
      # sets a description for it
      description: "the name of the person to greet",
      # set it as a required or optional argument
      required: true

    # define an option with a short flag using chars
    add_option 'c', "caps",
      # sets a description for it
      description: "greet with capitals",
      # set it as a required or optional flag
      required: false,
      # set whether it should take a value
      has_value: false,
      # optionally set a default value
      default: nil
  end
end
```

## Customising

The help template is divided into the following sections:
```
[HEADER]

[DESCRIPTION]

[USAGE]
    <NAME> <USE | "[<arguments>]" "[<options>]">

[COMMANDS]
    [ALIASES] <NAME> <SUMMARY>

[ARGUMENTS]
    <NAME> <DESCRIPTION> ["(required)"]

[OPTIONS]
    [SHORT] <LONG> <DESCRIPTION> ["(required)"] ["(default: ...)"]

[FOOTER]
```

Sections in `<>` will always be present, and ones in `[]` are optional depending on whether they are defined. Because of CLI.cr's modularity, this means that you could essentially have a blank help template (wouldn't recommend it though).

You can customise the following options for the help template formatter:
```crystal
class CLI::Formatter::Options
  # The character to use for flag option delimiters (default is `-`).
  property option_delim : Char

  # Whether to show the `default` tag for options with default values (default is `true`).
  property show_defaults : Bool

  # Whether to show the `required` tag for required arguments/options (default is `true`).
  property show_required : Bool
end
```

And pass it to the command like so:
```crystal
require "cli"

options = CLI::Formatter::Options.new option_delim: '+', show_defaults: false
# we can re-use this in multiple commands
formatter = CLI::Formatter.new options

class MainCommand < CLI::Command
  # ...

  def help_template : String
    formatter.generate self
  end
end
```

Alternatively, if you want a completely custom design, you can pass a string directly:
```crystal
def help_template : String
  <<-TXT
  My custom command help text!

  Use:
      greet <name> [-c | --caps] [-h | --help]
  TXT
end
```

## Motivation

Most Crystal CLI builders/DSLs are opinionated with limited customisation available. CLI.cr aims to be entirely modular so that you have the freedom to change whatever you want without having to write tons of boilerplate or monkey-patch code. Macro-based CLI shards can also be quite restrictive as they are not scalable, meaning that you may eventually have to refactor your application to another CLI shard. This is not meant to discourage you from using macro-based CLI shards, they are still useful for short and simple applications with a general template, but if you are looking for something to handle larger applications with guaranteed stability and scalability, CLI.cr is the library for you.

## Contributing
1. Fork it (<https://github.com/devnote-dev/cli.cr/fork>)
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request

## Contributors
- [Devonte W](https://github.com/devnote-dev) - creator and maintainer

This repository is managed under the Mozilla Public License v2.

?? 2022-present devnote-dev
