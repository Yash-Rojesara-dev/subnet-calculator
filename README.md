import random
import sys


def validate_ip(ip_address):
    ip_octets = ip_address.split(".")
    try:
        return (
            len(ip_octets) == 4
            and 1 <= int(ip_octets[0]) <= 223
            and int(ip_octets[0]) != 127
            and not (int(ip_octets[0]) == 169 and int(ip_octets[1]) == 254)
            and all(0 <= int(octet) <= 255 for octet in ip_octets)
        )
    except ValueError:
        return False


def validate_mask(mask):
    masks = [255, 254, 252, 248, 240, 224, 192, 128, 0]
    mask_octets = mask.split(".")
    try:
        return (
            len(mask_octets) == 4
            and int(mask_octets[0]) == 255
            and all(int(octet) in masks for octet in mask_octets)
            and int(mask_octets[0]) >= int(mask_octets[1]) >= int(mask_octets[2]) >= int(mask_octets[3])
        )
    except ValueError:
        return False


def to_binary(octets):
    return "".join(bin(int(octet))[2:].zfill(8) for octet in octets)


def calculate_subnet(ip, mask):
    ip_bin = to_binary(ip.split("."))
    mask_bin = to_binary(mask.split("."))

    host_bits = mask_bin.count("0")
    network_bin = ip_bin[:32 - host_bits] + "0" * host_bits
    broadcast_bin = ip_bin[:32 - host_bits] + "1" * host_bits

    def bin_to_ip(binary):
        return ".".join(str(int(binary[i:i+8], 2)) for i in range(0, 32, 8))

    network = bin_to_ip(network_bin)
    broadcast = bin_to_ip(broadcast_bin)
    hosts = abs(2 ** host_bits - 2)

    wildcard = ".".join(str(255 - int(o)) for o in mask.split("."))

    return network, broadcast, hosts, wildcard, 32 - host_bits


def generate_random_ip(network, broadcast):
    net = list(map(int, network.split(".")))
    bst = list(map(int, broadcast.split(".")))
    return ".".join(str(random.randint(net[i], bst[i])) for i in range(4))


def main():
    try:
        while True:
            ip = input("Enter IP address: ")
            if validate_ip(ip):
                break
            print("Invalid IP address. Try again.")

        while True:
            mask = input("Enter subnet mask: ")
            if validate_mask(mask):
                break
            print("Invalid subnet mask. Try again.")

        network, broadcast, hosts, wildcard, mask_bits = calculate_subnet(ip, mask)

        print("\nResults")
        print("Network Address:", network)
        print("Broadcast Address:", broadcast)
        print("Valid Hosts:", hosts)
        print("Wildcard Mask:", wildcard)
        print("Mask Bits:", mask_bits)

        while input("\nGenerate random IP? (y/n): ").lower() == "y":
            print("Random IP:", generate_random_ip(network, broadcast))

    except KeyboardInterrupt:
        print("\nProgram terminated.")
        sys.exit()


if __name__ == "__main__":
    main()
