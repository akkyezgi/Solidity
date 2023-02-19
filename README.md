# Solidity
# Piyango Akıllı Kontratı





// SPDX-License-Identifier: Unlicensed
pragma solidity ^0.8.4;

contract Lottery{
    address public manager; //public olarak kontratı yazan kişinin adresini tutar.
    address payable[] public players; //Katılımcılar para yatıracağı için payable yaptık. Yanına köşeli parantez koyduk ki listeleyebilelim.

    constructor() public {
        manager= msg.sender; // Başkaları bu kontrat üzerinden değişiklik yapamasın diye sahibini belli ediyoruz.
    }

    modifier onlyManager() { // Yani sadece kontratın sahibinin yapacağı işlemleri burada tutyoruz.
        require(msg.sender==manager, "Only manager can call this function");
        _;
    }

    //Events
    event playerInvested(address player, uint amount); // "amount" ne kaadar para yatırdığını gösterir.
    event winnerSelected(address winner, uint amount);

    //Invest Money (ellerindeki meblağnın ne kadar olduğunu bize göstereceki.)
    function invest() payable public {
        require(msg.sender!=manager, "Manager cannot invest"); // Burada kontratı yazan kişi katılamasın diye bu satırı yazdık.
        require(msg.value==13 ether, "You can only transfer 13 ether"); // Burada sadece 13 ether yatırma sınırı koyduk.
        players.push(payable(msg.sender)) ; //burada katılan herkesi oyunculara ekledik (push ile)
        emit playerInvested(msg.sender, msg.value); //burada da bir üstteki satırı kaydettik.
    }

    function getBalance() public view onlyManager returns(uint) { // Burada yatırılan tutarı sadece kontratı yazan 
    //kişinin görmesi için yazdık. Sadece kontratı yazan kişinin görmesini  de "onlyManager" ile sağladık
    return address(this).balance;
    }

    function random private view returns(uint) {
        return uint(keccak256(abi.encodePacked(block.timestamp, block.difficulty, players.length)));
    } // solidity`de random bir ifaade seçmek için bir kalıp olmadığından biz burada bir fonksiyon oluşturduk
    // en içteki paraantezdeki veriler sürekli değiştiği için bize random bir sayı verecek.

    function selectWinner() public onlyManager { // Burada kazananı tespit etmek için bir fonksiyon yazdık. 
    // onlyManager ile de kazananın sadece kontratı yazan kişinin görmesini sağladık.
        uint r=random();
        uint index=r%players.length;
        address payable winner=players[index];
        emit= winnerSelected(winner,address(this).balance);
        //41-44 arası satırlarda kazananı belirledik fakat ödülünü atmadık. Şimdi aşağıda ödülünü atacağız..
        winner.transfer(address(this).balance);
        players= new address payable[](0); // bu satırda kontrattaki adresleri temizledik ki bir daha ödül kazansınlar
    }

}
