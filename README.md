pragma solidity ^0.8.0;

import "./IERC20.sol";
import "./SafeMath.sol";
import "./Ownable.sol";

contract MyToken is IERC20, Ownable {
    using SafeMath for uint256;

    string public constant name = "MyToken";
    string public constant symbol = "MTK";
    uint8 public constant decimals = 18;

    mapping(address => uint256) private _balances;
    mapping(address => mapping(address => uint256)) private _allowances;

    uint256 private _totalSupply;

    // 锁仓规则
    mapping(address => uint256) private _lockedBalances;

    // 事件
    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);

    // 构造函数，设置总供应量并分发给合约创建者
    constructor(uint256 totalSupply_) {
        _totalSupply = totalSupply_ * 10 ** uint256(decimals);
        _balances[msg.sender] = _totalSupply;
        emit Transfer(address(0), msg.sender, _totalSupply);
    }

    // 获取总供应量
    function totalSupply() public view override returns (uint256) {
        return _totalSupply;
    }

    // 获取账户余额
    function balanceOf(address account) public view override returns (uint256) {
        return _balances[account];
    }

    // 转账
    function transfer(address recipient, uint256 amount) public override returns (bool) {
        _transfer(msg.sender, recipient, amount);
        return true;
    }

    // 授权
    function approve(address spender, uint256 amount) public override returns (bool) {
        _approve(msg.sender, spender, amount);
        return true;
    }

    // 转账授权
    function transferFrom(address sender, address recipient, uint256 amount) public override returns (bool) {
        _transfer(sender, recipient, amount);
        uint256 currentAllowance = _allowances[sender][msg.sender];
        require(currentAllowance >= amount, "ERC20: transfer amount exceeds allowance");
        _approve(sender, msg.sender, currentAllowance - amount);
        return true;
    }

    // 增加锁仓金额
    function lock(address account, uint256 amount) public onlyOwner {
        require(account != address(0), "ERC20: lock to the zero address");
        require(amount <= _balances[account], "ERC20: lock amount exceeds balance");
        _lockedBalances[account] = _lockedBalances[account].add(amount);
    }

    // 解锁金额
    function unlock(address account, uint256 amount) public onlyOwner {
        require(account != address(0), "ERC20: unlock from the zero address");
        require(amount <= _lockedBalances[account], "ERC20: unlock amount exceeds locked balance");
        _lockedBalances[account] = _lockedBalances[account].sub(amount);
    }

    // 获取锁定金额
    function lockedBalanceOf(address account) public view returns (uint256) {
        return _lockedBalances[account];
    }

    // 私有方法：转账函数
    function _transfer(address sender, address recipient, uint256 amount) private {
        require(sender != address(0), "ERC20: transfer from the zero address");
        require(recipient != address(0), "ERC20: transfer to the zero address");
        require(amount > 0, "ERC20: transfer amount must be greater than zero");
        require(_balances[sender] >= amount, "ERC20: transfer amount exceeds balance");
        require(_balances[recipient] + amount > _balances[recipient], "ERC20: transfer amount overflows");
        require(_lockedBalances[sender] <= _balances[sender] - amount, "ERC20: transfer amount exceeds unlocked balance");

        _balances[sender] = _balances[sender].sub(amount);
        _balances[recipient] = _balances[recipient].add(amount);
        emit Transfer(sender, recipient, amount);
    }

    // 私有方法：授权函数
    function _approve(address owner, address spender, uint256 amount) private {
        require(owner != address(0), "ERC20: approve from the zero address");
        require(spender != address(0), "ERC20: approve to the zero address");
        _allowances[owner][spender] = amount;
       
