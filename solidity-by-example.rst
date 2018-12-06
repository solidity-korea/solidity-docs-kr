###################
예제를 통한 솔리디티
###################

.. 색인:: voting, ballot

.. _voting:

******
투표
******

다음의 콘트렉트는 상당히 복잡하지만 solidity의 많은 특징들을 보여줍니다.
이는 투표에 관한 콘트렉트를 구현한 것입니다.
물론 전자 투표의 핵심 문제는 
어떻게 투표권을 올바르게 할당할것이며
이에 대한 조작을 어떻게 예방하느냐에 대한 것입니다.
여기서 모든 문제를 해결할 수 없겠지만 적어도 우리는
이 예제를 통해서 개표가 "자동적이고 동시에 완벽히 투명한지" 
위임투표가 어떻게 이루어지는지 보여줄 것입니다.


이 아이디어는 한개의 투표 당 하나의 컨트랙트를 작성하여 각각의 선택마다
짧은 이름을 제공합니다. 
그리고 의장 역할을 수행하는 컨트렉트 작성자는 
각 주소에 개별적인 투표권을 부여할 것입니다.

투표권을 받은 주소의 사람들은 
자신이 직접 투표하거나 자신의 투표권을 신뢰할 수 있는 다른 사람에게 위임할 수 있습니다.

투표 시간이 끝나면, ``winningProposal()``함수는 
가장 많은 득표를 한 안건을 반환할 것입니다.

::

    pragma solidity ^0.4.16;

    /// @title 위임 투표.
    contract Ballot {
        // 이것은 나중에 변수에 사용될 새로운
        // 복합 유형을 선언합니다.
        // 그것은 단일 유권자를 대표할 것입니다.
        struct Voter {
            uint weight; // weight 는 대표단에 의해 누적됩니다.
            bool voted;  // 만약 이 값이 true라면, 그 사람은 이미 투표한 것 입니다.
            address delegate; // 투표에 위임된 사람
            uint vote;   // 투표된 제안의 인덱스 데이터 값
        }

        // 이것은 단일 제안에 대한 유형입니다.
        struct Proposal {
            bytes32 name;   // 간단한 명칭 (최대 32바이트)
            uint voteCount; // 누적 투표 수
        }

        address public chairperson;

        // 이것은 각각의 가능한 주소에 대해
        // `Voter` 구조체를 저장하는 상태변수를 선언합니다.
        mapping(address => Voter) public voters;

        // 동적으로 크기가 지정된 `Proposal` 구조체의 배열입니다.
        Proposal[] public proposals;

        /// `proposalNames` 중 하나를 선택하기 위한 새로운 투표권을 생성십시오.
        function Ballot(bytes32[] proposalNames) public {
            chairperson = msg.sender;
            voters[chairperson].weight = 1;

            // 각각의 제공된 제안서 이름에 대해,
            // 새로운 제안서 개체를 만들어 배열 끝에 추가합니다.
            for (uint i = 0; i < proposalNames.length; i++) {
                // `Proposal({...})` creates a temporary
                // Proposal object and `proposals.push(...)`
                // appends it to the end of `proposals`.
                proposals.push(Proposal({
                    name: proposalNames[i],
                    voteCount: 0
                }));
            }
        }

        // `voter` 에게 이 투표권에 대한 권한을 부여하십시오.
        // 오직 `chairperson` 으로부터 호출받을 수 있습니다.
        function giveRightToVote(address voter) public {
            // `require`의 인수가 `false`로 평가되면,
            // 그것은 종료되고 모든 변경내용을 state와
            // Ether Balance로 되돌립니다.
            // 함수가 잘못 호출되면 이것을 사용하는 것이 좋습니다.
            // 그러나 조심하십시오,
            // 이것은 현재 제공된 모든 가스를 소비할 것입니다.
            // (이것은 앞으로 바뀌게 될 예정입니다)
            require(
                (msg.sender == chairperson) &&
                !voters[voter].voted &&
                (voters[voter].weight == 0)
            );
            voters[voter].weight = 1;
        }

        /// `to` 로 유권자에게 투표를 위임하십시오.
        function delegate(address to) public {
            // 참조를 지정하십시오.
            Voter storage sender = voters[msg.sender];
            require(!sender.voted);

            // 자체 위임은 허용되지 않습니다.
            require(to != msg.sender);

            // `to`가 위임하는 동안 delegation을 전달하십시오.
            // 일반적으로 이런 루프는 매우 위험하기 때문에,
            // 너무 오래 실행되면 블록에서 사용가능한 가스보다
            // 더 많은 가스가 필요하게 될지도 모릅니다.
            // 이 경우 위임(delegation)은 실행되지 않지만,
            // 다른 상황에서는 이러한 루프로 인해
            // 스마트 컨트랙트가 완전히 "고착"될 수 있습니다.
            while (voters[to].delegate != address(0)) {
                to = voters[to].delegate;

                // 우리는 delegation에 루프가 있음을 확인 했고 허용하지 않았습니다.
                require(to != msg.sender);
            }

            // `sender` 는 참조이므로,
            // `voters[msg.sender].voted` 를 수정합니다.
            sender.voted = true;
            sender.delegate = to;
            Voter storage delegate_ = voters[to];
            if (delegate_.voted) {
                // 대표가 이미 투표한 경우,
                // 투표 수에 직접 추가 하십시오
                proposals[delegate_.vote].voteCount += sender.weight;
            } else {
                // 대표가 아직 투표하지 않았다면,
                // weight에 추가하십시오.
                delegate_.weight += sender.weight;
            }
        }

        /// (당신에게 위임된 투표권을 포함하여)
        /// `proposals[proposal].name` 제안서에 투표 하십시오.
        function vote(uint proposal) public {
            Voter storage sender = voters[msg.sender];
            require(!sender.voted);
            sender.voted = true;
            sender.vote = proposal;

            // 만약 `proposal` 이 배열의 범위를 벗어나면
            // 자동으로 throw 하고 모든 변경사항을 되돌릴 것입니다.
            proposals[proposal].voteCount += sender.weight;
        }

        /// @dev 모든 이전 득표를 고려하여 승리한 제안서를 계산합니다.
        function winningProposal() public view
                returns (uint winningProposal_)
        {
            uint winningVoteCount = 0;
            for (uint p = 0; p < proposals.length; p++) {
                if (proposals[p].voteCount > winningVoteCount) {
                    winningVoteCount = proposals[p].voteCount;
                    winningProposal_ = p;
                }
            }
        }

        // winningProposal() 함수를 호출하여
        // 제안 배열에 포함된 승자의 index를 가져온 다음
        // 승자의 이름을 반환합니다.
        function winnerName() public view
                returns (bytes32 winnerName_)
        {
            winnerName_ = proposals[winningProposal()].name;
        }
    }


개선가능 한 사안들
=====================

현재 모든 참가자에게 투표권을 부여하기 위해 많은 거래가 필요 합니다.
더 나은 방법을 생각해 볼 수 있습니까?

.. 색인:: auction;blind, auction;open, blind auction, open auction

*************
블라인드 경매
*************

이 섹션에서는, 이더리움 블라인드 경매 콘트렉트를 완전하게 만드는것이 얼마나
쉬운지 보여줄 것입니다.
우리는 누구나 제시되어진 가격을 볼수 있는 공개 경매로 시작하여 이 콘트렉트를 
입찰기간이 끝나기전까지 실제 입찰가를 보는 것이 불가능한 블라인드 경매로
확장시킬것입니다.


.. _simple_auction:

간단한 공개 경매
===================

아래의 간단한 경매 콘트랙트에 대한 일반적인 아이디어는 경매 기간동안 모든 사람이
그들의 가격제시를 전송할 수 있다는 것입니다. 가격 제시는 이미 가격제시자(bidder)와
그들의 가격제시(bid)를 묶기위해서 돈이나 이더를 송금하는 것을
포함하고 있습니다. 만약 최고 제시 가격이 올라간다면, 그 이전의 최고 제시자는 그녀의 돈을
돌려받습니다. 가격제시 기간이 끝난후, 그 콘트렉트는 그의 돈을 받는 수혜자를 위해 
수동으로 호출 되어져야만 합니다 - 콘트렉트는 그들 스스로 활성화 시킬 수 없습니다.


:

    pragma solidity ^0.4.21;

    contract SimpleAuction {
        // 옥션의 파라미터. 시간은 아래 둘중 하나입니다.
        // 앱솔루트 유닉스 타임스탬프 (seconds since 1970-01-01)
        // 혹은 시한(time period) in seconds.
        address public beneficiary;
        uint public auctionEnd;

        // 옥션의 현재 상황.
        address public highestBidder;
        uint public highestBid;

        // 이전 가격 제시들의 수락된 출금.
        mapping(address => uint) pendingReturns;

        // 마지막에 true 로 설정, 어떠한 변경도 허락되지 않습니다.
        bool ended;

        // 변경에 발생하는 이벤트
        event HighestBidIncreased(address bidder, uint amount);
        event AuctionEnded(address winner, uint amount);

        // 아래의 것은 소위 "natspec"이라고 불리우는 코멘트,
        // 3개의 슬래시에 의해 알아볼 수 있습니다.
        // 이것을 유저가 트렌젝션에 대한 확인을 요청 받을때
        // 보여집니다.

        /// 수혜자의 주소를 대신하여 두번째 가격제시 기간 '_biddingTime'과
        /// 수혜자의 주소 '_beneficiary' 를 포함하는
        /// 간단한 옥션을 제작합니다.
        function SimpleAuction(
            uint _biddingTime,
            address _beneficiary
        ) public {
            beneficiary = _beneficiary;
            auctionEnd = now + _biddingTime;
        }

        /// 경매에 대한 가격제시와 값은
        /// 이 transaction과 함께 보내집니다.
        /// 값은 경매에서 이기지 못했을 경우만
        /// 반환 받을 수 있습니다.
        function bid() public payable {
            // 어떤 인자도 필요하지 않음, 모든
            // 모든 정보는 이미 트렌젝션의
            // 일부이다. 'payable' 키워드는
            // 이더를 지급는 것이 가능 하도록
            // 하기 위하여 함수에게 요구됩니다. 

            // 경매 기간이 끝났으면
            // 되돌아 갑니다.
            require(now <= auctionEnd);

            // 만약 이 가격제시가 더 높지 않다면, 돈을 
            // 되돌려 보냅니다.
            require(msg.value > highestBid);

            if (highestBid != 0) {
                // 간단히 highestBidder.send(highestBid)를 사용하여
                // 돈을 돌려 보내는 것은 보안상의 리스크가 있습니다.
                // 그것은 신뢰되지 않은 콘트렉트를 실행 시킬수 있기 때문입니다.
                // 받는 사람이 그들의 돈을 그들 스스로 출금 하도록 하는 것이
                // 항상 더 안전합니다.
                pendingReturns[highestBidder] += highestBid;
            }
            highestBidder = msg.sender;
            highestBid = msg.value;
            emit HighestBidIncreased(msg.sender, msg.value);
        }

        /// 비싸게 값이 불러진 가격제시 출금.
        function withdraw() public returns (bool) {
            uint amount = pendingReturns[msg.sender];
            if (amount > 0) {
                // 받는 사람이 이 `send` 반환 이전에 받는 호출의 일부로써
                // 이 함수를 다시 호출할 수 있기 때문에
                // 이것을 0으로 설정하는 것은 중요하다.
                pendingReturns[msg.sender] = 0;

                if (!msg.sender.send(amount)) {
                    // 여기서 throw를 호출할 필요가 없습니다, 빚진 양만 초기화.
                    pendingReturns[msg.sender] = amount;
                    return false;
                }
            }
            return true;
        }

        /// 이 경매를 끝내고 최고 가격 제시를 
        /// 수혜자에게 송금.
        function auctionEnd() public {

            // 이것은 다른 콘트렉트와 상호작용하는 함수의 구조를 잡는 좋은 가이드 라인입니다. 
            // (i.e. 그것들은 이더를 보내거나 함수를 호출합니다.)
            // 3가지 단계:
            // 1. 조건을 확인
            // 2. 동작을 수행  (잠재적으로 변경되는 조건)
            // 3. interacting with other contracts
            // If these phases are mixed up, the other contract could call
            // back into the current contract and modify the state or cause
            // effects (ether payout) to be performed multiple times.
            // If functions called internally include interaction with external
            // contracts, they also have to be considered interaction with
            // external contracts.

            // 1. 조건
            require(now >= auctionEnd); // auction did not yet end
            require(!ended); // this function has already been called

            // 2. 영향
            ended = true;
            emit AuctionEnded(highestBidder, highestBid);

            // 3. 상호작용
            beneficiary.transfer(highestBid);
        }
    }

블라인드 경매
=============

이전의 공개 경매는 블라인드 경매로 확장 되었습니다.
블라인드 경매의 이점은 경매 기간 마감에 대한 시간 압박이
없다는 것입니다. 블라인드 경매를 트렌젝션 컴퓨팅 플렛폼에 만드는
것은 모순 되는 말 처럼 들릴지도 모르지만, 그러나 암호학이
이를 구조했습니다. 


**경매 기간** 동안, 가격 제시자들은 실제로 그녀의 가격제시를
보내는 것이 아니라, 그 버전의 해쉬 버전입니다.
두 해쉬 값이 같은 두개의 (충분히 긴)값을 찾는 것은 현재 실용적으로
불가능 하다고 여겨지기 때문에, 가격 제시자들은 그것에 의해 가격을 제시합니다.
경매 기간이 끝난 후, 가격 제시자들은 그들이 가격을 제시한 것들을 드러내야만 
합니다: 그들은 그들의 암호화되지 않은 값을 보내고 콘트렉트는 해쉬 값이 경매
기간동안 제공되어진 것과 같은지 확인합니다.

또다른 도전은 어떻게 경매를 **묶고 블라인드**로 동시에 만드는가 입니다
: 가격 제시자를 그가 경매에서 이긴 후에 돈을 보내는 것을 막는
유일한 방법은 그녀가 돈을 가격 제시와 함께 보내도록 하는 것입니다.
값 전달이 이더리움 안에서 블라인드가 될 수 없기 때문에, 누구든 그
값을 볼 수 있습니다.

아래의 콘트렉트는 이 문제를 가장 큰 가격제시보다 더큰 어떤 값이든 수락
함으로써 해결했습니다. 이것은 당연히 드러내는 단계에서만 확인되어 질 수 있기
때문에, 몇몇 가격 제시는 **유효하지 않을**지도 모릅니다, 그리고 이것은 고의
입니다. (그것은 심지어 높은 가치의 이전과 함께 유효하지 않은 입찰에 배치할 명시적인
깃발을 제공합니다.): 가격 제시자들은 몇몇의 높고 낮은 유효하지 않은 가격 제시를
함으로써 경쟁을 혼란 스럽게 할 수 있습니다.

::

    pragma solidity ^0.4.21;

    contract BlindAuction {
        struct Bid {
            bytes32 blindedBid;
            uint deposit;
        }

        address public beneficiary;
        uint public biddingEnd;
        uint public revealEnd;
        bool public ended;

        mapping(address => Bid[]) public bids;

        address public highestBidder;
        uint public highestBid;

        // 이전 가격 제시의 허락된 출금
        mapping(address => uint) pendingReturns;

        event AuctionEnded(address winner, uint highestBid);

        /// Modifier는 함수 입력값을 입증하는 편리한 방법입니다.
        /// `onlyBefore`은 아래의 'bid'에 적용 되어 질 수 있습니다:
        /// 이 새로운 함수 몸체는 `_`이 오래된 함수 몸체를 대체하는
        /// Modifier의 몸체 입니다.
        modifier onlyBefore(uint _time) { require(now < _time); _; }
        modifier onlyAfter(uint _time) { require(now > _time); _; }

        function BlindAuction(
            uint _biddingTime,
            uint _revealTime,
            address _beneficiary
        ) public {
            beneficiary = _beneficiary;
            biddingEnd = now + _biddingTime;
            revealEnd = biddingEnd + _revealTime;
        }

        /// `_blindedBid` = keccak256(value, fake, secret)와 함께
        /// 가려진(blinded) 가격을 제시합니다.
        /// 만약 가격 제시가 드러내는 단계에서 올바르게 보여진다면
        /// 보내진 이더는 환급받을 수 만 있습니다. 
        /// 가격 제시와 함께 보내진 이더는 적어도 "value"와"fake" 는 참입니다.
        /// "fake"를 참으로 설정하고 정확하지 않은 양을 보내는 것은 진짜 가격 제시를
        /// 숨기는 방법들입니다. 그러나 여전히 요구되는 출금을 합니다. 같은
        /// 주소는 여러 가격 제시들을 둘 수 있습니다.
        function bid(bytes32 _blindedBid)
            public
            payable
            onlyBefore(biddingEnd)
        {
            bids[msg.sender].push(Bid({
                blindedBid: _blindedBid,
                deposit: msg.value
            }));
        }

        /// Reveal your blinded bids. You will get a refund for all
        /// correctly blinded invalid bids and for all bids except for
        /// the totally highest.
        /// 가려진 가격 제시를 드러냅니다. 너는 알맞게 가려진 유효하지 않은
        /// 가격 제시들을 되돌려 받을 것입니다. 그리고 가장 높은 가격 제시를 제외하고
        /// 모든 가격 제시도 돌려 받을 것입니다.
        function reveal(
            uint[] _values,
            bool[] _fake,
            bytes32[] _secret
        )
            public
            onlyAfter(biddingEnd)
            onlyBefore(revealEnd)
        {
            uint length = bids[msg.sender].length;
            require(_values.length == length);
            require(_fake.length == length);
            require(_secret.length == length);

            uint refund;
            for (uint i = 0; i < length; i++) {
                var bid = bids[msg.sender][i];
                var (value, fake, secret) =
                        (_values[i], _fake[i], _secret[i]);
                if (bid.blindedBid != keccak256(value, fake, secret)) {
                    // 가격 제시는 실제로 드러나지 않습니다.
                    // Do not refund deposit.
                    continue;
                }
                refund += bid.deposit;
                if (!fake && bid.deposit >= value) {
                    if (placeBid(msg.sender, value))
                        refund -= value;
                }
                // Make it impossible for the sender to re-claim
                // the same deposit.
                bid.blindedBid = bytes32(0);
            }
            msg.sender.transfer(refund);
        }

        // 이것은 이 함수가 이 콘트렉트 안에서 이것 스스로만 호출 될 수
        // 있다는 의미를 가지는 "internal" 함수입니다.
        // (혹은 파생된 콘트렉트들에서)
        function placeBid(address bidder, uint value) internal
                returns (bool success)
        {
            if (value <= highestBid) {
                return false;
            }
            if (highestBidder != 0) {
                // 이전에 가장 높은 가격 제시를 환급
                pendingReturns[highestBidder] += highestBid;
            }
            highestBid = value;
            highestBidder = bidder;
            return true;
        }

        /// Withdraw a bid that was overbid.
        function withdraw() public {
            uint amount = pendingReturns[msg.sender];
            if (amount > 0) {
                // It is important to set this to zero because the recipient
                // can call this function again as part of the receiving call
                // before `transfer` returns (see the remark above about
                // conditions -> effects -> interaction).
                pendingReturns[msg.sender] = 0;

                msg.sender.transfer(amount);
            }
        }

        /// 경매를 끝내고 가장 높은 가격 제시를 수혜자에게
        /// 송금합니다.
        function auctionEnd()
            public
            onlyAfter(revealEnd)
        {
            require(!ended);
            emit AuctionEnded(highestBidder, highestBid);
            ended = true;
            beneficiary.transfer(highestBid);
        }
    }


.. index:: purchase, remote purchase, escrow

********************
Safe Remote Purchase
********************

::

    pragma solidity ^0.4.21;

    contract Purchase {
        uint public value;
        address public seller;
        address public buyer;
        enum State { Created, Locked, Inactive }
        State public state;

        // `msg.value`가 짝수임을 확실히 하세요.
        // 만약 홀수라면 분할은 길이를 줄일 것입니다. 
        // 곱셈을 통해 이것이 홀수가 아닌지 확인하세요.
        function Purchase() public payable {
            seller = msg.sender;
            value = msg.value / 2;
            require((2 * value) == msg.value);
        }

        modifier condition(bool _condition) {
            require(_condition);
            _;
        }

        modifier onlyBuyer() {
            require(msg.sender == buyer);
            _;
        }

        modifier onlySeller() {
            require(msg.sender == seller);
            _;
        }

        modifier inState(State _state) {
            require(state == _state);
            _;
        }

        event Aborted();
        event PurchaseConfirmed();
        event ItemReceived();

        /// 구매를 중단하고 이더를 회수합니다.
        /// 콘트렉트가 잠기기 전에 판매자에 의해서만
        /// 호출 되어 질 수 있습니다.
        function abort()
            public
            onlySeller
            inState(State.Created)
        {
            emit Aborted();
            state = State.Inactive;
            seller.transfer(this.balance);
        }

        /// 구매자로서 구매를 확정합니다.
        /// 트렌젝션은 `2 * value` ether 을 포함해야 한다.
        /// 이 이더는 confirmReceived()가 호출 될때까지
        /// 잠길것입니다.
        function confirmPurchase()
            public
            inState(State.Created)
            condition(msg.value == (2 * value))
            payable
        {
            emit PurchaseConfirmed();
            buyer = msg.sender;
            state = State.Locked;
        }

        /// 구매자가 아이템을 받았다고 확인.
        /// 이것은 잠긴 이더를 풀어줄것입니다.
        function confirmReceived()
            public
            onlyBuyer
            inState(State.Locked)
        {
            emit ItemReceived();
            // It is important to change the state first because
            // otherwise, the contracts called using `send` below
            // can call in again here.
            // 먼저 상태를 변경하는 것이 중요합니다.
            // 그렇지 않으면, `send`를 사용하며 호출된 콘트렉트는
            // 다시 여기를 호출할 수 있기 때문입니다.
            state = State.Inactive;

            // NOTE: 이것은 실제로 구매자와 판매자 둘다 환급 하는 것을 막을 수 
            // 있도록 합니다. - 출금 패턴이 사용되어야만 합니다.

            buyer.transfer(value);
            seller.transfer(this.balance);
        }
    }

********************
Micropayment Channel
********************

앞으로 쓰여질 예정
