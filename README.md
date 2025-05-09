import React, { useState, useEffect } from 'react';
import { motion } from 'framer-motion';
import { Card, CardContent } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import axios from 'axios';

const NotCoinClone = () => {
    const [balance, setBalance] = useState(0);
    const [clickValue, setClickValue] = useState(1);
    const [floatingCoins, setFloatingCoins] = useState([]);
    const [autoClickerOwned, setAutoClickerOwned] = useState(false);
    const [autoClickerActive, setAutoClickerActive] = useState(false);
    const [autoClickerTimer, setAutoClickerTimer] = useState(0);
    const [level, setLevel] = useState(1);
    const [subscribed, setSubscribed] = useState(false);
    const [leaderboard, setLeaderboard] = useState([]);
    const [socialLinks, setSocialLinks] = useState({ telegram: '', twitter: '' });
    const [lastClaimedBonus, setLastClaimedBonus] = useState(null);
    const [maxClicks, setMaxClicks] = useState(1000);
    const [clicksLeft, setClicksLeft] = useState(1000);
    const [clickResetTimer, setClickResetTimer] = useState(0);

    const TON_WALLET = "UQCvhujOBIxkeEYE63tOZqY1yh9vrNoeLKsHTG8RxO1II36y";
    const TON_REWARD = 5000;
    const DAILY_BONUS = 100;
    const CLICK_RESET_TIME = 3 * 60 * 60; // 3 hours

    useEffect(() => {
        fetchLeaderboard();
        fetchSocialLinks();
        loadBonusStatus();
        setMaxClicks(1000 + (level - 1) * 300);
        setClicksLeft(1000 + (level - 1) * 300);
        startClickResetTimer();
    }, [level]);

    useEffect(() => {
        const interval = setInterval(() => {
            if (clickResetTimer > 0) {
                setClickResetTimer(clickResetTimer - 1);
            } else {
                setClicksLeft(1000 + (level - 1) * 300);
                setClickResetTimer(CLICK_RESET_TIME);
            }
        }, 1000);
        return () => clearInterval(interval);
    }, [clickResetTimer]);

    const startClickResetTimer = () => {
        const lastReset = localStorage.getItem('lastClickReset');
        const now = Math.floor(Date.now() / 1000);

        if (lastReset) {
            const elapsed = now - parseInt(lastReset);
            const remaining = Math.max(CLICK_RESET_TIME - elapsed, 0);
            setClickResetTimer(remaining);
            if (remaining === 0) {
                setClicksLeft(1000 + (level - 1) * 300);
                localStorage.setItem('lastClickReset', now.toString());
            }
        } else {
            setClickResetTimer(CLICK_RESET_TIME);
            localStorage.setItem('lastClickReset', now.toString());
        }
    };

    const formatTime = (seconds) => {
        const hours = Math.floor(seconds / 3600);
        const minutes = Math.floor((seconds % 3600) / 60);
        const secs = seconds % 60;
        return `${hours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;
    };

    const handleCoinClick = () => {
        if (clicksLeft > 0) {
            setBalance(balance + clickValue);
            setClicksLeft(clicksLeft - 1);
            const newCoin = { id: Date.now() };
            setFloatingCoins([...floatingCoins, newCoin]);
            setTimeout(() => {
                setFloatingCoins(currentCoins => currentCoins.filter(c => c.id !== newCoin.id));
            }, 1000);
        } else {
            alert('⚠️ Достигнут максимальный лимит кликов. Подождите, пока счётчик сбросится.');
        }
    };

    const handleDonateClick = () => {
        window.open(`https://tonhub.com/transfer/${TON_WALLET}?amount=1000000000`, '_blank');
    };

    return (
        <div className="flex flex-col items-center justify-center min-h-screen bg-gradient-to-b from-purple-900 via-blue-800 to-indigo-900 text-white p-4">
            <motion.div 
                className="mb-8 w-72 h-72 bg-yellow-500 rounded-full flex items-center justify-center shadow-2xl cursor-pointer" 
                whileTap={{ scale: 0.9 }}
                onClick={handleCoinClick}
            >
                <h1 className="text-4xl font-bold text-white">WHERE TOKEN</h1>
            </motion.div>
            <p className="text-xl text-gray-300 mb-4">Осталось кликов: {clicksLeft} / {maxClicks}</p>
            <p className="text-lg text-gray-400 mb-6">До сброса: {formatTime(clickResetTimer)}</p>
            <div className="flex gap-4">
                <Button className="flex-1 h-20 bg-red-600 text-white font-bold text-lg rounded-lg shadow-lg hover:bg-red-500" onClick={() => alert('Открывается таблица лидеров')}>
                    🏆 Лидеры
                </Button>
                <Button className="flex-1 h-20 bg-green-600 text-white font-bold text-lg rounded-lg shadow-lg hover:bg-green-500" onClick={handleDonateClick}>
                    💰 $1 = 5000 токенов
                </Button>
            </div>
        </div>
    );
};

export default NotCoinClone;
