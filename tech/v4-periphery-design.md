
# DeltaResolver
这段代码是一个抽象合约 DeltaResolver，用于同步、发送和结算资金到 PoolManager。它主要处理与资金转移相关的功能，并且通过调用 PoolManager 中的相应函数来管理代币的余额和结算。

主要功能和含义：
	1.	错误定义：
	•	DeltaNotPositive: 当尝试结算一个正的 delta 时触发，表示该操作不可进行，因为余额应为负数。
	•	DeltaNotNegative: 当尝试结算一个负的 delta 时触发，表示该操作不可进行，因为余额应为正数。
	•	InsufficientBalance: 当合约余额不足以进行包装或解包装操作时触发。
	2.	主要函数：
	•	_take：
	•	从 PoolManager 中提取一定数量的货币，并发送到指定的接收者地址。
	•	如果金额为零，则直接返回。
	•	_settle：
	•	结算某个货币的余额到 PoolManager。
	•	调用 poolManager.sync(currency) 来同步该货币。
	•	如果是 ETH（通过 currency.isAddressZero() 判断），则直接通过 settle{value: amount}() 进行结算；否则，通过 _pay 函数将代币转账到 PoolManager。
	•	_pay：
	•	一个抽象函数，由实现该合约的具体合约提供代币支付逻辑。此函数会向 PoolManager 支付代币。
	•	_getFullDebt：
	•	获取合约对某个货币的全部债务。如果债务是正数，则无法结算（应该是取出）。通过调用 poolManager.currencyDelta(address(this), currency) 来获取当前合约的 delta 值。
	•	_getFullCredit：
	•	获取合约对某个货币的全部信用额度。如果信用是负数，则无法取出（应该是结算）。通过调用 poolManager.currencyDelta(address(this), currency) 来获取当前合约的 delta 值。
	•	_mapSettleAmount：
	•	计算结算操作的金额。根据传入的 amount，如果是 CONTRACT_BALANCE，则返回当前合约的余额；如果是 OPEN_DELTA，则返回全部债务金额；否则返回指定的金额。
	•	_mapTakeAmount：
	•	计算取出操作的金额。根据传入的 amount，如果是 OPEN_DELTA，则返回全部信用金额；否则返回指定的金额。
	•	_mapWrapUnwrapAmount：
	•	计算包装（wrap）或解包装（unwrap）操作的金额。根据输入货币和输出货币的类型，决定是从合约余额中取出金额，还是从 delta 中获取金额。
	•	如果金额为 CONTRACT_BALANCE，则直接返回合约的余额。
	•	如果金额为 OPEN_DELTA，则根据债务金额来决定包装或解包装的操作金额。
	•	如果余额不足，则触发 InsufficientBalance 错误。

总结：

DeltaResolver 合约是一个为与 PoolManager 进行资金结算、同步和操作的抽象合约。它提供了操作资金的基本框架和逻辑，具体的支付代币和如何操作资金（如包装、解包装等）则由继承该合约的具体合约来实现。通过这种方式，合约可以灵活地管理不同类型的货币和资金流动。