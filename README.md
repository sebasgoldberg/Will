# Will
Contract to execute Will initiated by beneficiary and forwarded by witnesses.


## Idea

The idea here is very simple:
- We have beneficiaries with priorities assigned to them.
- When a beneficiary activates the Will, it convert himself in the `main beneficiary`, and a time-lock is activated (`activation stage`).
- After time-lock is released, then the contract can be executed by the beneficiary that activated the time lock (`release stage`).
- After execution the `beneficiary` of the contract will be the `owner` of the contract, and it returns to `initial stage`.
- During `activation stage` or `release stage`, another beneficiary with higher priority can activate again the Will, and convert himself in the `main beneficiary`. If this happens a time-lock is activated again.
- During the `activation stage`, the release of the time-lock can be forwarded an specific amount of `forward time` by the `witnesses` (other beneficiaries that are not the `main beneficiary`).

## Implementation

### Ownable
The contract will be an [Ownable](https://docs.openzeppelin.com/contracts/4.x/access-control#ownership-and-ownable) contract:

### Configuration
 - `beneficiaries` with their corresponding priority ({address, uint8}[]).
 - `timeLockAmountOfTime` after will activation (uint256).
 - `forwardAmountOfTime` (uint256)

### Status
- `mainBeneficiary` (address)
- `activationDate` The initial date where the contract can be executed. (uint256)
- `forwardsByWitness` The witness that already forwarded the `activation stage` (address\<bool>).
- `locked` Beneficiary operations are locked.

### Operations
#### Owner Operations
- `addBeneficiary(address beneficiary, uint8 priority)` (if `beneficiary` exists, then change its priority)
- `removeBeneficiary(address beneficiary)` (if `mainBeneficiary` == `beneficiary`, then first call to `goToInicialStage()`)
- `setTimeLockAmount(uint256 amount)`
- `setForwardTimeAmount(uint256 amount)`
- `goToInicialStage()`
- `lock()`
- `unlock()`

#### Beneficiary Operations

- `activate()` 
	- Requires:
		- Not activated by another beneficiary with higher priority.
		- Contract is in `initial stage`
		- `not locked`
	- Result:
		- Sets the `mainBeneficiary`
		- Sets the `activationDate`
		- Clear the `forwardsByWitness`

- `forward()`
	-  Requires:
		- `sender != mainBeneficiary`: Who is calling the forward it is not the main beneficiary.
		- Contract is in `activation stage`
		- `not forwardsByWitness[sender]`: Who is calling the forward, not called before.
		- `not locked`
	- Result
		- Update the `activationDate`

- `execute()` 
	- Requires
		- Contract is in `execution stage` (time-lock released)
		- `sender == mainBeneficiary`
		- `not locked`
	- Result
		- Calls `doExecute`
		- Sets `mainBeneficiary` as the contract's `owner`
		- Calls `goToInicialStage()`
		- Calls `removeBeneficiary(mainBeneficiary)`

#### Execution
- `doExecute` (abstract operation to be defined)
