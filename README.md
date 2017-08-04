# Joker

using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace Joker
{
    public enum CardSuit
    {
        Heart, //3
        Diamond,//4
        Club,//5
        Spade//6
    }

    public enum CardValue
    {
        Two = 2,
        Three,
        Four,
        Five,
        Six,
        Seven,
        Eight,
        Nine,
        Ten,
        Jack,
        Queen,
        King,
        Ace
    }

    class Program
    {
        static void Main(string[] args)
        {
            GameEngine g = new GameEngine();
            g.Start();
        }
    }

    class GameEngine
    {
       public Deck deck { get; set; }

        public GameEngine()
        {
            deck = new Deck();
            deck.Shuffle();
        }

        public void Start()
        {
            Player p =  new Player(deck.DealHand());
            p.DisplayHand();
            p.DisplaySortedHand();
            p.DisplayPosibilities();
            Console.Read();

        }

    }

    class Deck
    {
        public List<Card> Cards { get; set; }

        public Deck()
        {
            Cards = new List<Card>();

            foreach (CardSuit color  in Enum.GetValues(typeof(CardSuit)))
            {
                foreach (CardValue value in Enum.GetValues(typeof(CardValue)))
                {
                    Cards.Add(new Card(color, value));
                    Cards.Add(new Card(color, value));
                }
            }
        }

        public void Shuffle()
        {
            Random r = new Random();

            for (int i = 0; i < 1000; i++)
            {
                int from = r.Next(0, Cards.Count);
                int to = r.Next(0, Cards.Count);
                Card _c = Cards[from];
                Cards.RemoveAt(from);
                    Cards.Insert(to, _c);
            }

            foreach (Card c in Cards)
            {
                Debug.WriteLine(c.Color + " " + c.Value);
            }
        }

        public List<Card> DealHand()
        {
            List<Card> list = new List<Card>();
            Random r = new Random();
            for (int i = 1; i <= 14; i++)
            {
                int _i = r.Next(0, Cards.Count);
                list.Add(Cards[_i]);
                Cards.RemoveAt(_i);
            }
            return list;
        }



    }

    public class Card
    {
        public CardSuit Color { get; set; }

        public CardValue Value { get; set; }

        public Card(CardSuit color, CardValue value)
        {
            this.Color = color;
            this.Value = value;
        }

        public override string ToString()
        {
            string retStr = string.Empty;

            if (Value == CardValue.Ace)
                retStr += 'A';
            else retStr += (int)Value;

            switch (Color)
            {
                case CardSuit.Heart: retStr += (char)3;break;
                case CardSuit.Diamond: retStr += (char)4; break;
                case CardSuit.Club: retStr += (char)5; break;
                case CardSuit.Spade: retStr += (char)6; break;
            }

            return retStr;
        }
    }

    class Player
    {
        private CardCollection PlayerCardCollection {  get; set; }
        public List<Card> Hand { get; set; }

        public Player(List<Card> dealtCards)
        {
            PlayerCardCollection = new CardCollection();
            PlayerCardCollection.AddCardsWhenDealHand(dealtCards);
            Hand = dealtCards;
        }

        public void DisplayHand(List<Card> cards = null)
        {
            List<Card> _cards = cards == null ? Hand : cards;

            for (int i = 0; i <= 13; i++)
            {
                Console.WriteLine("{0:D2}. {1,3}", i + 1, _cards[i]);
            }
        }

        public void DisplaySortedHand()
        {
            List<Card> cards = new List<Card>();
            foreach (CardSuit suit in Enum.GetValues(typeof(CardSuit)))
            {
                foreach (CardValue value in Enum.GetValues(typeof(CardValue)))
                {
                    if (PlayerCardCollection.cardCollection[suit][value] == 0) { }
                    else 
                    {
                        if (PlayerCardCollection.cardCollection[suit][value] == 2)
                            cards.Add(new Card(suit, value));
                        cards.Add(new Card(suit, value));
                    }
                }


            }
            DisplayHand(cards);


        }

        public void DisplayPosibilities()
        {
            List<List<Card>> CardRuns = new List<List<Card>>();
            //check for runs
            foreach (CardSuit c in Enum.GetValues(typeof(CardSuit)))
            {
                if (PlayerCardCollection.cardCollection[c].Where(_ => _.Value > 0).Count() > 3)
                {

                    for (int i = 1; i <= 14;)
                    {
                        bool endOfRun = false;
                        int runLength = 0;
                        {
                            while (!endOfRun && i <= 14)
                            {
                                if (PlayerCardCollection.cardCollection[c][(CardValue)(i == 1 ? 14 : i)] > 0)
                                {
                                    runLength++;
                                    i++;
                                }
                                else
                                {
                                    endOfRun = true;
                                }

                            }
                            if (runLength >= 3)
                            {
                                List<Card> run = new List<Card>();
                                for (int j = -(i+1); j < 0; j++)
                                {
                                    run.Add(new Card(c,(CardValue)j));
                                }
                                CardRuns.Add(run);
                            }
                        }
                    }
                }
            }
        }
    }

    public class CardCollection
    {
        public Dictionary<CardSuit,Dictionary<CardValue,int>> cardCollection { get; set; }

        public CardCollection()
        {
            cardCollection = new Dictionary<CardSuit, Dictionary<CardValue, int>>();

            foreach (CardSuit cardColor in Enum.GetValues(typeof(CardSuit)))
            {
                cardCollection.Add(cardColor, new Dictionary<CardValue, int>());
                for (int i = 2; i <= 14; i++)
                    cardCollection[cardColor].Add((CardValue)i, 0);
            }
        }

        public void AddCardsWhenDealHand(List<Card> cards)
        {
            foreach (Card card in cards)
                cardCollection[card.Color][card.Value] +=1;
        }

    }
}
