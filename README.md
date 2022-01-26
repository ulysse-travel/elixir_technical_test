# Ulysse Technical Test

## Purpose
The purpose of this exercise is to test out your capacity to solve a problem. It's possible that you have never coded in Elixir nor a functional programming language before but don't worry, we will take this into account.

NB: If you have never worked on Elixir, it is also a good opportunity to know more about the language itself and realize if you want to work with it on a daily basis @ Ulysse.

## General acceptance criteria
- All code must be in a git repo (you can use your own GitHub account).
- You can use any Elixir or Erlang library you want to.
- Code accuracy also matters. A readable, safe, refactorable code is a plus.

## The project
Using Phoenix, you are going to implement a simpler version of the Ulysse flight search feature that can be found at https://ulysse.com.

At Ulysse, flights search uses multiple providers. One of them is [Duffel](https://duffel.com). 

In order to simplify this exercise, let's say that we can only search for one-way tickets for an adult.

1. Create a new Phoenix project named `homer` :
`mix phx.new homer`

2. Use the following command to create the `Search` context and the `OfferRequest` schema : 
`mix phx.gen.context Search OfferRequest offer_requests origin:string destination:string departure_date:date sort_by:string allowed_airlines:array:string`

Replace the following in `Homer.Search.OfferRequest`:
```diff
- field :sort_by, :string
+ field :sort_by, Ecto.Enum, values: [:total_amount, :total_duration]
```

3. Modify the generated code to ensure that the following specs are respected :
- We can create an offer request by providing these fields (and only these one): origin, destination and a departure date.
- Origin and destination are airport iata code.
- All airlines are allowed by default.
- Offers are sorted by price by default (ascending order).
- We can only update the list of allowed airlines and the sort order.

4. Use the following command to create the `Offer` schema : `mix phx.gen.schema Search.Offer offers origin:string destination:string departing_at:naive_datetime arriving_at:naive_datetime segments_count:integer total_amount:decimal total_duration:integer`

PS: `total_duration` is in minutes

5. Create the `Homer.Search.Duffel` module that must implement the following behaviour :
```elixir
defmodule Homer.Search.ProviderBehaviour do
  alias Homer.Search.{OfferRequest, Offer}
  
  @callback fetch_offers(OfferRequest.t()) :: {:ok, [Offer.t()]} | {:error, any()}
end
```

6. Create the `Homer.Search.Server` module which use a [GenServer](https://hexdocs.pm/elixir/1.12/GenServer.html) and that must implement the following behaviour :
```elixir
defmodule Homer.Search.ServerBehaviour do
  @callback start_link({OfferRequest.t(), providers :: [module()]}) :: {:ok, pid()} | {:error, any()}

  @doc """
  Get the filtered and sorted list of offers.
  """
  @callback list_offers(pid :: pid(), limit :: integer()) :: {:ok, [Offer.t()]} | {:error, any()}

  @doc """
  Notify the server that an offer request has been updated.
  """
  @callback offer_request_updated(pid :: pid(), OfferRequest.t()) :: :ok
end
```

7. Modify the `Homer.Search` module to implement to following behaviour by using what you did in the previous steps. You can add as many functions as you want.
```elixir
defmodule Homer.Search.Behaviour do
  @callback list_offer_requests() :: [OfferRequest.t()]

  @callback get_offer_request!(id :: integer()) :: [OfferRequest.t()]

  @callback create_offer_request(attrs :: map()) :: {:ok, OfferRequest.t()} | {:error, any()}

  @callback update_offer_request(OfferRequest.t(), attrs :: map()) :: {:ok, OfferRequest.t()} | {:error, any()}

  @callback get_offers(OfferRequest.t(), limit :: integer()) :: {:ok, [Offer.t()]} | {:error, any()}
end
```
