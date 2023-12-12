# BLACKJACK
This C++ program simulates a simplified version of the classic casino card game, Blackjack. Developed by Jesse Martinez, this console-based application allows users to play Blackjack against a computerized dealer. The program incorporates fundamental Blackjack rules, including card dealing, player decisions (hit or stand), and determining the winner based on hand values. It also includes a basic heuristic to estimate winning probabilities for the player.

#include <iostream>
#include <vector>
#include <algorithm>
#include <ctime>
#include <cstdlib>

enum Rank { TWO, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT, NINE, TEN, JACK, QUEEN, KING, ACE };
enum Suit { HEARTS, DIAMONDS, CLUBS, SPADES };

struct Card {
    Rank rank;
    Suit suit;
};

class Deck {
private:
    std::vector<Card> cards;

public:
    Deck() {
        // Initialize the deck with 52 cards
        for (int suit = HEARTS; suit <= SPADES; ++suit) {
            for (int rank = TWO; rank <= ACE; ++rank) {
                cards.push_back({ static_cast<Rank>(rank), static_cast<Suit>(suit) });
            }
        }
        // Shuffle the deck
        std::random_shuffle(cards.begin(), cards.end());
    }

    Card drawCard() {
        Card card = cards.back();
        cards.pop_back();
        return card;
    }
};

class Hand {
private:
    std::vector<Card> cards;

public:
    void addCard(Card card) {
        cards.push_back(card);
    }

    int calculateHandValue() {
        int value = 0;
        int numAces = 0;

        for (const auto& card : cards) {
            if (card.rank == ACE) {
                ++numAces;
            }
            value += (card.rank >= TWO && card.rank <= TEN) ? static_cast<int>(card.rank) + 2 : 10;
        }

        // Adjust for Aces
        while (numAces > 0 && value + 10 <= 21) {
            value += 10;
            --numAces;
        }

        return value;
    }

    void showHand(bool showAll = true) {
        std::cout << "Hand: ";
        for (const auto& card : cards) {
            if (showAll) {
                std::cout << card.rank << " ";
            } else {
                std::cout << "X ";
            }
        }
        std::cout << "\n";
    }
};

class BlackjackGame {
private:
    Deck deck;
    Hand dealerHand;
    Hand playerHand;

public:
    void play() {
        // Initial deal
        playerHand.addCard(deck.drawCard());
        dealerHand.addCard(deck.drawCard());
        playerHand.addCard(deck.drawCard());
        dealerHand.addCard(deck.drawCard());

        // Show initial hands
        playerHand.showHand();
        dealerHand.showHand(false);

        // Calculate winning probability for the player's hand
        double initialProbability = calculateWinningProbability(playerHand.calculateHandValue());

        std::cout << "Initial Winning Probability: " << initialProbability << "%\n";

        // Allow the player to hit or stand
        char choice;
        do {
            std::cout << "Do you want to hit? (y/n): ";
            std::cin >> choice;

            if (choice == 'y') {
                playerHand.addCard(deck.drawCard());
                playerHand.showHand();
                // Calculate winning probability after hitting
                double probabilityAfterHit = calculateWinningProbability(playerHand.calculateHandValue());
                std::cout << "Winning Probability after Hit: " << probabilityAfterHit << "%\n";
            }

        } while (choice == 'y');

        // Dealer's turn
        while (dealerHand.calculateHandValue() < 17) {
            dealerHand.addCard(deck.drawCard());
        }

        // Determine the winner
        determineWinner();
    }

double calculateWinningProbability(int handValue) {
    // Basic heuristic: Assume the player has a 40% chance of winning
    // and adjust based on the player's hand value
    double baseProbability = 40.0;

    // Adjust the probability based on the player's hand value
    if (handValue > 21) {
        return 0.0;  // Player busted, no chance of winning
    } else if (handValue >= 17 && handValue <= 21) {
        return 80.0;  // High probability of winning with a good hand
    } else {
        return baseProbability;
    }
}

    void determineWinner() {
        // Determine and display the winner based on hand values
        int playerValue = playerHand.calculateHandValue();
        int dealerValue = dealerHand.calculateHandValue();

        std::cout << "Player's hand value: " << playerValue << "\n";
        std::cout << "Dealer's hand value: " << dealerValue << "\n";

        if (playerValue > 21 || (dealerValue <= 21 && dealerValue >= playerValue)) {
            std::cout << "Dealer wins!\n";
        } else {
            std::cout << "Player wins!\n";
        }
    }
};

int main() {
    // Seed the random number generator
    std::srand(std::time(0));

    // Play the Blackjack game
    BlackjackGame game;
    game.play();

    return 0;
}
