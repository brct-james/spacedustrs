# spacedustrs
This is a rust API wrapper for https://spacetraders.io V2

Many of the client functions in this project are based on, or in some cases copied directly from, the work of https://github.com/bloveless/spacetraders-rs

## Quickstart

Use the following example to get started. 

```rust
use spacedustrs::client::Client;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    env_logger::init();

    // Setup Game Client
    let http_client = spacedustrs::client::get_http_client(None);

    // Register agent
    let claim_agent_response = spacedustrs::client::claim_agent(
        http_client,
        "https://v2-0-0.alpha.spacetraders.io".to_string(),
        "<4-8 character string>".to_string(),
        "COMMERCE_REPUBLIC".to_string(),
    )
    .await.unwrap();

    // Setup client using claimed agent
    let client = Client::new(
        http_client,
        "https://v2-0-0.alpha.spacetraders.io".to_string(),
        claim_agent_response.data.agent.symbol,
        claim_agent_response.token,
    );

    // Get agent details to confirm working
    match client.get_my_agent_details().await {
        Ok(res) => {
            println!("{:#?}", res);
        }
        Err(res_err) => {
            println!("{:#?}", res_err);
        }
    }

    Ok(())
}
```

## Supported Endpoints

### Public Endpoints

**Agent & Account**

- `POST` /agents
- - `spacedustrs::client::claim_agent(base_url: String, http_client: http_client, agent_symbol: String, agent_faction: String) -> responses::ClaimAgent`

### Non-Specific Endpoints That Still Require Authentication

**Systems**

- `GET` /systems
- - `Client.get_systems() -> responses::SystemsListResponse`
- `GET` /systems/{systemSymbol}
- - `Client.get_system_info(system_symbol: String) -> responses::SystemInformationResponse`
- `GET` /systems/{systemSymbol}/waypoints
- - `Client.get_system_waypoints(system_symbol: String) -> responses::SystemWaypointsResponse`
- `GET` /systems/{systemSymbol}/waypoints/{waypointSymbol}
- - `Client.get_system_waypoint(system_symbol: String, waypoint_symbol: String) -> responses::SystemWaypointResponse`
- `GET` /systems/{systemSymbol}/shipyards # view all shipyards in a system
- - `Client.get_system_shipyards(system_symbol: String) -> responses::SystemShipyardsResponse`
- `GET` /systems/{systemSymbol}/shipyards/{shipyardSymbol}
- - `Client.get_system_shipyard(system_symbol: String, shipyard_symbol: String) -> responses::SystemShipyardResponse`
- `GET` /systems/{systemSymbol}/shipyards/{shipyardSymbol}/ships # view all ships for sell at a shipyard
- - `Client.get_shipyard_ships(system_symbol: String, shipyard_symbol: String) -> responses::ShipyardShipsResponse`
- `GET` /systems/{systemSymbol}/markets # view all markets in a system
- - `Client.get_system_markets(system_symbol: String) -> responses::SystemMarketsResponse`
- `GET` /systems/{systemSymbol}/markets/{marketSymbol} # view all trades at a given market
- - `Client.get_system_market(system_symbol: String, market_symbol: String) -> responses::SystemMarketResponse`

### Agent Specific Endpoints

**Agent**

- `GET` /my/agent
- - `Client.get_my_agent_details() -> responses::AgentDetails`

**Contracts**

- `GET` /my/contracts
- - `Client.get_my_contracts() -> responses::ContractsResponse`
- `GET` /my/contracts/{contractId}
- - `Client.get_my_contract(contract_id: String) -> responses::ContractResponse`
- `POST` /my/contracts/{contractId}/accept
- - `Client.get_my_contract(contract_id: String) -> responses::AcceptedContractResponse`

**Ships**

- `GET` /my/ships
- - `Client.get_my_ships() -> responses::ShipsResponse`
- `GET` /my/ships/{shipSymbol}
- - `Client.get_my_ship(ship_id: String) -> responses::ShipResponse`
- `POST` /my/ships/{shipSymbol}/chart
- - `Client.chart(ship_id: String) -> responses::ChartResponse`
- `GET` /my/ships/{shipSymbol}/navigate
- - `Client.ship_navigation_status(ship_id: String) -> responses::NavigateResponse`
- `POST` /my/ships/{shipSymbol}/navigate destination=destination_symbol
- - `Client.ship_navigation_status(ship_id: String, destination_symbol: String) -> responses::NavigateResponse`
- `GET` /my/ships/{shipSymbol}/survey
- - `Client.get_survey_cooldown(ship_id: String) -> responses::CooldownResponse`
- `POST` /my/ships/{shipSymbol}/survey
- - `Client.survey_surroundings(ship_id: String) -> responses::SurveyResponse`
- `GET` /my/ships/{shipSymbol}/extract
- - `Client.get_extract_cooldown(ship_id: String) -> responses::CooldownResponse`
- `POST` /my/ships/{shipSymbol}/extract ?survey{}=survey
- - `Client.extract_resources(ship_id: String, survey: Option<Survey>) -> responses::ExtractResourcesResponse`
- `POST` /my/ships/{shipSymbol}/dock
- - `Client.dock_ship(ship_id: String) -> responses::StatusResponse`
- `POST` /my/ships/{shipSymbol}/orbit
- - `Client.orbit_ship(ship_id: String) -> responses::StatusResponse`
- `POST` /my/ships/{shipSymbol}/deliver
- - `Client.deliver_goods(ship_id: String, contract_id: String, trade_symbol: String, units: u64) -> responses::DeliveryResponse`
- `POST` /my/ships/{shipSymbol}/refuel
- - `Client.refuel_ship(ship_id: String) -> responses::RefuelResponse`
- `POST` /my/ships/{shipSymbol}/scan mode=scan_mode
- - `Client.scan_ships(ship_id: String, mode: shared::ScanMode) -> responses::ScanResponse`
- `POST` /my/ships/{shipSymbol}/jettison symbol=HEAVY_MACHINERY units=99999
- - `Client.jettison_cargo(ship_id: String, trade_symbol: String, units: u64) -> responses::TransactionResponse`
- `POST` /my/ships/{shipSymbol}/sell symbol=HEAVY_MACHINERY units=99999
- - `Client.sell_cargo(ship_id: String, trade_symbol: String, units: u64) -> responses::TransactionResponse`
- `POST` /my/ships/{shipSymbol}/purchase symbol=HEAVY_MACHINERY units=99999
- - `Client.buy_cargo(ship_id: String, trade_symbol: String, units: u64) -> responses::TransactionResponse`
- `GET` /my/ships/{shipSymbol}/jump
- - `Client.get_jump_cooldown(ship_id: String) -> responses::CooldownResponse`
- `POST` /my/ships/{shipSymbol}/jump destination=X1-OE
- - `Client.jump(ship_id: String, destination: String) -> responses::JumpResponse`

## Unsupported Endpoints

- ships/Jump, Deploy, 
- trade/symbol/import, export, exchange (appears to lookup what markets are importing/exporting/exchanging that good?)
- contracts/deliver
- POST: /my/ships (purchase)
- scan modes: APPROACHING_SHIPS DEPARTING_SHIPS SYSTEM WAYPOINT

## Notes from Roadmap

- `/navigate` will support modes
- `/dock` will support modes
- `/orbit` will support modes
- `/systems/.../markets` may eventually have commodities
- `GET` /my/account # not implemented
- `GET` /my/ships/{shipSymbol}/scan # not implemented but should return cooldown
- `POST` /my/ships/{shipSymbol}/chart # appears to be incomplete
- Once `agent/delete` comes online, each test should create its own agent and check several functions for its lifetime, allowing multiple threads to work simultaneously and ensuring state for state-dependent endpoints

## TODO

- Review changes to jump, extract, survey and scan endpoints
- Add purchase ship