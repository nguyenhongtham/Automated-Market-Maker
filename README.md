# Automated-Market-Maker
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.26;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract BaseAMMv2 is Ownable {
    IERC20 public token0;
    IERC20 public token1;
    uint256 public reserve0;
    uint256 public reserve1;
    uint256 public totalLiquidity;
    mapping(address => uint256) public liquidity;

    event Swap(address indexed sender, uint256 amount0In, uint256 amount1Out);

    constructor(address _token0, address _token1) {
        token0 = IERC20(_token0);
        token1 = IERC20(_token1);
    }

    function addLiquidity(uint256 amount0, uint256 amount1) external {
        token0.transferFrom(msg.sender, address(this), amount0);
        token1.transferFrom(msg.sender, address(this), amount1);

        uint256 liquidityMinted = sqrt(amount0 * amount1);
        liquidity[msg.sender] += liquidityMinted;
        totalLiquidity += liquidityMinted;

        reserve0 += amount0;
        reserve1 += amount1;
    }

    function swap(uint256 amount0In, uint256 amount1In) external {
        require(amount0In > 0 || amount1In > 0, "No input");

        uint256 amount0Out = (amount1In * reserve0) / (reserve1 + amount1In);
        uint256 amount1Out = (amount0In * reserve1) / (reserve0 + amount0In);

        if (amount0In > 0) token0.transferFrom(msg.sender, address(this), amount0In);
        if (amount1In > 0) token1.transferFrom(msg.sender, address(this), amount1In);

        if (amount0Out > 0) token0.transfer(msg.sender, amount0Out);
        if (amount1Out > 0) token1.transfer(msg.sender, amount1Out);

        reserve0 = token0.balanceOf(address(this));
        reserve1 = token1.balanceOf(address(this));

        emit Swap(msg.sender, amount0In, amount1Out);
    }

    function sqrt(uint256 y) internal pure returns (uint256 z) {
        z = y;
        uint256 x = y / 2 + 1;
        while (x < z) {
            z = x;
            x = (y / x + x) / 2;
        }
    }
}
