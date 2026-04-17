# base123import time
from collections import Counter
from web3 import Web3

RPC_URL = "https://mainnet.base.org"

WINDOW_BLOCKS = 20
PAIR_THRESHOLD = 5  # alert if same pair interacts this many times


def main():
    w3 = Web3(Web3.HTTPProvider(RPC_URL))

    if not w3.is_connected():
        raise RuntimeError("Cannot connect to Base RPC")

    print("Connected to Base")
    print("Tracking repeated wallet interactions...\n")

    last_block = w3.eth.block_number

    while True:
        try:
            current_block = w3.eth.block_number

            if current_block >= last_block + WINDOW_BLOCKS:

                from_block = current_block - WINDOW_BLOCKS
                to_block = current_block

                pair_counter = Counter()

                for b in range(from_block, to_block + 1):

                    block = w3.eth.get_block(b, full_transactions=True)

                    for tx in block.transactions:

                        if tx["to"]:
                            pair = (tx["from"], tx["to"])
                            pair_counter[pair] += 1

                print(f"\nBlocks {from_block} → {to_block}")

                for (sender, receiver), count in pair_counter.items():
                    if count >= PAIR_THRESHOLD:
                        print("🔁 Repeated Interaction")
                        print("From:", sender)
                        print("To:", receiver)
                        print("Count:", count)
                        print()

                last_block = current_block

            time.sleep(3)

        except Exception as e:
            print("Error:", e)
            time.sleep(5)


if __name__ == "__main__":
    main()
