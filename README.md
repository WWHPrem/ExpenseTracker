import { useState } from 'react'
import { Button } from "/components/ui/button"
import { Card, CardContent, CardHeader, CardTitle, CardDescription } from "/components/ui/card"
import { Input } from "/components/ui/input"
import { Label } from "/components/ui/label"
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "/components/ui/select"
import { Trash, ArrowDown, ArrowUp } from "lucide-react"

type Transaction = {
  id: string
  description: string
  amount: number
  category: string
  date: string
  type: 'income' | 'expense'
}

// Format currency for Indian Rupees
const formatRupees = (amount: number) => {
  return new Intl.NumberFormat('en-IN', {
    style: 'currency',
    currency: 'INR',
    minimumFractionDigits: 0,
    maximumFractionDigits: 2
  }).format(amount)
}

export default function IncomeExpenseTracker() {
  const [transactions, setTransactions] = useState<Transaction[]>([])
  const [description, setDescription] = useState('')
  const [amount, setAmount] = useState('')
  const [category, setCategory] = useState('')
  const [transactionType, setTransactionType] = useState<'income' | 'expense'>('expense')
  const [filterCategory, setFilterCategory] = useState('all')
  const [filterType, setFilterType] = useState<'all' | 'income' | 'expense'>('all')

  // Indian-specific categories
  const incomeCategories = [
    'Salary',
    'Freelance',
    'Investments',
    'Rental Income',
    'Business',
    'Gifts',
    'Other Income'
  ]

  const expenseCategories = [
    'Groceries',
    'Transportation',
    'Entertainment',
    'Utilities',
    'Shopping',
    'Rent',
    'EMI',
    'Education',
    'Healthcare',
    'Other Expense'
  ]

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault()
    
    if (!description || !amount || !category) {
      alert('Please fill all fields')
      return
    }

    const amountValue = parseFloat(amount)
    if (isNaN(amountValue) || amountValue <= 0) {
      alert('Please enter a valid amount')
      return
    }

    const newTransaction: Transaction = {
      id: Date.now().toString(),
      description,
      amount: amountValue,
      category,
      date: new Date().toLocaleDateString('en-IN'),
      type: transactionType
    }

    setTransactions([...transactions, newTransaction])
    setDescription('')
    setAmount('')
    setCategory('')
  }

  const deleteTransaction = (id: string) => {
    setTransactions(transactions.filter(transaction => transaction.id !== id))
  }

  const filteredTransactions = transactions.filter(transaction => {
    const categoryMatch = filterCategory === 'all' || transaction.category === filterCategory
    const typeMatch = filterType === 'all' || transaction.type === filterType
    return categoryMatch && typeMatch
  })

  const totalIncome = transactions
    .filter(t => t.type === 'income')
    .reduce((sum, transaction) => sum + transaction.amount, 0)

  const totalExpenses = transactions
    .filter(t => t.type === 'expense')
    .reduce((sum, transaction) => sum + transaction.amount, 0)

  const balance = totalIncome - totalExpenses

  const currentCategories = transactionType === 'income' ? incomeCategories : expenseCategories

  return (
    <div className="max-w-2xl mx-auto p-4 space-y-6">
      <Card>
        <CardHeader>
          <CardTitle>Income & Expense Tracker</CardTitle>
          <CardDescription>Track your financial transactions in Indian Rupees</CardDescription>
        </CardHeader>
        <CardContent>
          <form onSubmit={handleSubmit} className="space-y-4">
            <div className="flex gap-2">
              <Button
                type="button"
                variant={transactionType === 'income' ? 'default' : 'outline'}
                onClick={() => setTransactionType('income')}
              >
                <ArrowUp className="h-4 w-4 mr-2" />
                Income
              </Button>
              <Button
                type="button"
                variant={transactionType === 'expense' ? 'default' : 'outline'}
                onClick={() => setTransactionType('expense')}
              >
                <ArrowDown className="h-4 w-4 mr-2" />
                Expense
              </Button>
            </div>

            <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
              <div className="space-y-2">
                <Label htmlFor="description">Description</Label>
                <Input
                  id="description"
                  value={description}
                  onChange={(e) => setDescription(e.target.value)}
                  placeholder={transactionType === 'income' ? "Income source" : "What did you spend on?"}
                />
              </div>
              <div className="space-y-2">
                <Label htmlFor="amount">Amount (₹)</Label>
                <Input
                  id="amount"
                  type="number"
                  value={amount}
                  onChange={(e) => setAmount(e.target.value)}
                  placeholder="0.00"
                  step="1"
                />
              </div>
              <div className="space-y-2">
                <Label htmlFor="category">Category</Label>
                <Select value={category} onValueChange={setCategory}>
                  <SelectTrigger>
                    <SelectValue placeholder="Select category" />
                  </SelectTrigger>
                  <SelectContent>
                    {currentCategories.map((cat) => (
                      <SelectItem key={cat} value={cat}>
                        {cat}
                      </SelectItem>
                    ))}
                  </SelectContent>
                </Select>
              </div>
            </div>
            <Button type="submit" className="w-full md:w-auto">
              Add {transactionType === 'income' ? 'Income' : 'Expense'}
            </Button>
          </form>
        </CardContent>
      </Card>

      <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
        <Card>
          <CardHeader className="pb-2">
            <CardDescription>Balance</CardDescription>
            <CardTitle className={`text-3xl ${
              balance >= 0 ? 'text-green-600' : 'text-red-600'
            }`}>
              {formatRupees(balance)}
            </CardTitle>
          </CardHeader>
        </Card>
        <Card>
          <CardHeader className="pb-2">
            <CardDescription>Income</CardDescription>
            <CardTitle className="text-3xl text-green-600">
              {formatRupees(totalIncome)}
            </CardTitle>
          </CardHeader>
        </Card>
        <Card>
          <CardHeader className="pb-2">
            <CardDescription>Expenses</CardDescription>
            <CardTitle className="text-3xl text-red-600">
              {formatRupees(totalExpenses)}
            </CardTitle>
          </CardHeader>
        </Card>
      </div>

      <Card>
        <CardHeader>
          <div className="flex flex-col md:flex-row md:justify-between md:items-center gap-4">
            <div>
              <CardTitle>Your Transactions</CardTitle>
              <CardDescription>
                Showing {filteredTransactions.length} transactions
              </CardDescription>
            </div>
            <div className="grid grid-cols-1 md:grid-cols-2 gap-2">
              <Select value={filterType} onValueChange={setFilterType}>
                <SelectTrigger>
                  <SelectValue placeholder="Filter by type" />
                </SelectTrigger>
                <SelectContent>
                  <SelectItem value="all">All Types</SelectItem>
                  <SelectItem value="income">Income</SelectItem>
                  <SelectItem value="expense">Expenses</SelectItem>
                </SelectContent>
              </Select>
              <Select value={filterCategory} onValueChange={setFilterCategory}>
                <SelectTrigger>
                  <SelectValue placeholder="Filter by category" />
                </SelectTrigger>
                <SelectContent>
                  <SelectItem value="all">All Categories</SelectItem>
                  {[...incomeCategories, ...expenseCategories].map((cat) => (
                    <SelectItem key={cat} value={cat}>
                      {cat}
                    </SelectItem>
                  ))}
                </SelectContent>
              </Select>
            </div>
          </div>
        </CardHeader>
        <CardContent>
          {filteredTransactions.length === 0 ? (
            <p className="text-center text-gray-500 py-4">
              No transactions to show. Add your first transaction!
            </p>
          ) : (
            <div className="space-y-2">
              {filteredTransactions.map((transaction) => (
                <div
                  key={transaction.id}
                  className={`flex justify-between items-center p-3 border rounded-lg ${
                    transaction.type === 'income' ? 'bg-green-50' : 'bg-red-50'
                  }`}
                >
                  <div className="flex items-center gap-3">
                    <div className={`p-2 rounded-full ${
                      transaction.type === 'income' 
                        ? 'bg-green-100 text-green-600' 
                        : 'bg-red-100 text-red-600'
                    }`}>
                      {transaction.type === 'income' ? (
                        <ArrowUp className="h-4 w-4" />
                      ) : (
                        <ArrowDown className="h-4 w-4" />
                      )}
                    </div>
                    <div>
                      <p className="font-medium">{transaction.description}</p>
                      <div className="flex gap-4 text-sm text-gray-500">
                        <span>{transaction.category}</span>
                        <span>{transaction.date}</span>
                      </div>
                    </div>
                  </div>
                  <div className="flex items-center gap-2">
                    <span className={`font-medium ${
                      transaction.type === 'income' ? 'text-green-600' : 'text-red-600'
                    }`}>
                      {transaction.type === 'income' ? '+' : '-'}{formatRupees(transaction.amount)}
                    </span>
                    <Button
                      variant="ghost"
                      size="icon"
                      onClick={() => deleteTransaction(transaction.id)}
                    >
                      <Trash className="h-4 w-4 text-gray-500" />
                    </Button>
                  </div>
                </div>
              ))}
            </div>
          )}
        </CardContent>
      </Card>
    </div>
  )
}
