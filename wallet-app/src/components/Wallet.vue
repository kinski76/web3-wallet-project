<template>
    <!-- Template remains the same -->
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