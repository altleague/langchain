# LangChain

**NOTE**: This project is under active development and is subject to significant changes.

**LangChain** is a framework for developing applications powered by language models. It enables applications that are:

- **Data-aware:** connect a language model to other sources of data
- **Agentic:** allow a language model to interact with its environment

The main value props of LangChain are:

1. **Components:** abstractions for working with language models, along with a collection of implementations for each abstraction. Components are modular and easy-to-use, whether you are using the rest of the LangChain framework or not
1. **Off-the-shelf chains:** a structured assembly of components for accomplishing specific higher-level tasks

Off-the-shelf chains make it easy to get started. For more complex applications and nuanced use-cases, components make it easy to customize existing chains or build new ones.

## What is this?

Large Language Models (LLMs) are emerging as a transformative technology, enabling developers to build applications that they previously could not. But using these LLMs in isolation is often not enough to create a truly powerful app - the real power comes when you can combine them with other sources of computation or knowledge.

This library is aimed at assisting in the development of those types of applications.

## Documentation

The online documentation can [found here](https://hexdocs.pm/langchain).

## Relationship with JavaScript and Python LangChain

This library is written in [Elixir](https://elixir-lang.org/) and intended to be used with Elixir applications. The original libraries are [LangChain JS](https://js.langchain.com/) and [LangChain Python](https://python.langchain.com/).

The JavaScript and Python projects aim to integrate with each other as seamlessly as possible. The intended integration is so strong that that all objects (prompts, LLMs, chains, etc) are designed in a way where they can be serialized and shared between the two languages.

This Elixir version does not aim for parity with the JavaScript and Python libraries. Why not?

- JavaScript and Python are both Object Oriented languages. Elixir is Functional. We're not going to force a design that doesn't apply.
- The JS and Python versions started before conversational LLMs were standard. They put a lot of effort into preserving history (like a conversation) when the LLM didn't support it. We're not doing that here.

This library was heavily inspired by, and based on, the way the JavaScript library actually worked and interacted with an LLM.

## Installation

The package can be installed by adding `langchain` to your list of dependencies
in `mix.exs`:

```elixir
def deps do
  [
    {:langchain, "~> 0.1.0"}
  ]
end
```

## Configuration

Currently, the library is written to use the `Req` library for making API calls.

To make API calls, it is necessary to configure the API keys for the services you connect with. At this time, the library only supports ChatGPT and the OpenAI API key.

```elixir
import Config

config :langchain, openai_key: System.get_env("OPENAI_KEY")
# OR
config :langchain, openai_key: "YOUR SECRET KEY"
```

It's possible to use a function or a tuple to resolve the secret:

```elixir
config :langchain, openai_key: {MyApp.Secrets, :openai_key, []}
# OR
config :langchain, openai_key: fn -> System.get_env("OPENAI_KEY") end
```

When doing local development on the `Langchain` library itself, rename the `.envrc_template` to `.envrc` and populate it with your private API values. This is only used when running live test when explicitly requested.

Use a tool like [Dotenv](https://github.com/motdotla/dotenv) to load the API values into the ENV when using the library locally.

## Usage


### Exposing a custom Elixir function to ChatGPT

```elixir
alias Langchain.Function
alias Langchain.Message
alias Langchain.Chains.LLMChain
alias Langchain.ChatModels.ChatOpenAI

# map of data we want to be passed as `context` to the function when
# executed.
custom_context = %{
  "user_id" => 123,
  "hairbrush" => "drawer",
  "dog" => "backyard",
  "sandwich" => "kitchen"
}

# a custom Elixir function made available to the LLM
custom_fn =
  Function.new!(%{
    name: "custom",
    description: "Returns the location of the requested element or item.",
    parameters_schema: %{
      type: "object",
      properties: %{
        thing: %{
          type: "string",
          description: "The thing whose location is being requested."
        }
      },
      required: ["thing"]
    },
    function: fn %{"thing" => thing} = _arguments, context ->
      # our context is a pretend item/location location map
      context[thing]
    end
  })

# create and run the chain
{:ok, updated_chain, %Message{} = message} =
  LLMChain.new!(%{
    llm: ChatOpenAI.new!(),
    custom_context: custom_context,
    verbose: true
  })
  |> LLMChain.add_functions(custom_fn)
  |> LLMChain.add_message(Message.new_user!("Where is the hairbrush located?"))
  |> LLMChain.run(while_needs_response: true)

# print the LLM's answer
IO.put message.content
#=> "The hairbrush is located in the drawer."
```

## Testing

To run all the tests including the ones that perform live calls against the OpenAI API, use the following command:

```
mix test --include live_call
```

NOTE: This will use the configured API credentials which creates billable events.

Otherwise, running the following will only run local tests making no external API calls:

```
mix test
```

Executing a specific test, wether it is a `live_call` or not, will execute it creating a potentially billable event.