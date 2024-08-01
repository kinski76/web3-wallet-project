<template>
    <div class="wallet p-6 md:p-12 max-w-lg mx-auto bg-white rounded-xl shadow-2xl space-y-4">
        <div class="wallet-item">
            <label for="public-key" class="wallet-label block text-sm font-medium text-gray-700">Public Key:</label>
            <div id="public-key" class="wallet-field mt-1 block w-full p-2 border border-gray-300 rounded-md">{{
                publicKey }}</div>
        </div>
        <div class="wallet-item w-full">
            <label for="balance" class="wallet-label block text-sm font-medium text-gray-700">Balance:</label>
            <div id="balance" class="wallet-field mt-1 block w-full p-2 border border-gray-300 rounded-md">{{ balance }}
                ETH</div>
        </div>
        <div class="wallet-item w-full flex justify-center">
            <w3m-button />
        </div>
        <button v-if="isConnected" @click="sendTransaction"
            class="send-button w-full py-2 px-4 border border-transparent text-sm font-medium rounded-2xl text-white bg-green-600 hover:bg-green-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-green-500">
            Send Transaction
        </button>
    </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { ethers } from 'ethers';
import { createWeb3Modal, defaultConfig, useWeb3ModalState, useWeb3ModalProvider, useWeb3ModalAccount } from '@web3modal/ethers/vue';

const publicKey = ref<string>('');
const balance = ref<string>('0');
const wallet = ref<ethers.HDNodeWallet | null>(null);

const projectId = import.meta.env.VITE_PROJECT_ID as string;

interface Chain {
    chainId: number;
    name: string;
    currency: string;
    explorerUrl: string;
    rpcUrl: string;
}

const chains: Chain[] = [
    {
        chainId: 2730,
        name: 'XR Sepolia',
        currency: 'tXR',
        explorerUrl: 'https://etherscan.io',
        rpcUrl: 'https://xr-sepolia-testnet.rpc.caldera.xyz/http'
    }
];

const metadata = {
    name: 'wallet app',
    description: 'test wallet app',
    url: 'https://mywebsite.com',
    icons: ['https://avatars.mywebsite.com/']
};

const ethersConfig = defaultConfig({ metadata });

createWeb3Modal({
    ethersConfig,
    chains,
    projectId,
    enableAnalytics: true
});

const { isConnected } = useWeb3ModalAccount();
const { walletProvider } = useWeb3ModalProvider();

const generateWallet = (): void => {
    const newWallet = ethers.Wallet.createRandom();
    if (!(newWallet instanceof ethers.HDNodeWallet)) {
        throw new Error("Failed to create HDNodeWallet");
    }
    wallet.value = newWallet;
    publicKey.value = newWallet.address;
    localStorage.setItem('privateKey', newWallet.privateKey);
    console.log('Wallet generated:', newWallet);
    balance.value = '0'; // Reset balance for new wallet
    getBalance();
};

const getBalance = async (): Promise<void> => {
    if (!wallet.value) return;
    try {
        const provider = new ethers.JsonRpcProvider(chains[0].rpcUrl);
        const bal = await provider.getBalance(wallet.value.address);
        balance.value = ethers.formatEther(bal);
    } catch (error) {
        console.error('Error fetching balance:', error);
    }
};

const sendTransaction = async (): Promise<void> => {
    if (!walletProvider.value && !wallet.value) return;
    try {
        let signer: ethers.Signer;
        if (walletProvider.value) {
            const provider = new ethers.BrowserProvider(walletProvider.value);
            signer = await provider.getSigner();
        } else if (wallet.value) {
            const provider = new ethers.JsonRpcProvider(chains[0].rpcUrl);
            signer = wallet.value.connect(provider);
        } else {
            throw new Error("No wallet available");
        }
        const tx = await signer.sendTransaction({
            to: publicKey.value,
            value: ethers.parseEther('0.01')
        });
        await tx.wait();
        console.log('Transaction sent:', tx.hash);
        getBalance();
    } catch (error) {
        console.error('Error sending transaction:', error);
    }
};

onMounted(() => {
    generateWallet();
});
</script>
