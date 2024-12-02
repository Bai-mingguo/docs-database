datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

model User {
  id                 Int                @id @default(autoincrement())
  username           String             @unique
  email              String             @unique
  passwordHash       String
  fullName           String
  dateOfBirth        DateTime
  country            String
  createdAt          DateTime           @default(now())
  lastLogin          DateTime?
  accountStatus      AccountStatus      @default(ACTIVE)
  verificationStatus VerificationStatus @default(UNVERIFIED)
  roleId             Int
  role               Role               @relation(fields: [roleId], references: [id])
  accounts           Account[]
  userRiskParameter  UserRiskParameter?
  kycAmlData         KYC_AML_Data?
  auditLogs          AuditLog[]
  orders             Order[]
  positions          Position[]
}

model Role {
  id          Int     @id @default(autoincrement())
  name        String  @unique
  description String?
  users       User[]
}

model Account {
  id           Int          @id @default(autoincrement())
  userId       Int
  user         User         @relation(fields: [userId], references: [id])
  accountType  AccountType
  currency     String
  balance      Decimal      @default(0)
  createdAt    DateTime     @default(now())
  lastUpdated  DateTime     @updatedAt
  transactions Transaction[]
  marginUsage  MarginUsage[]
  orders       Order[]
  positions    Position[]
}

model Instrument {
  id                Int       @id @default(autoincrement())
  symbol            String    @unique
  name              String
  type              InstrumentType
  pipValue          Decimal
  minTradeSize      Decimal
  maxTradeSize      Decimal
  marginRequirement Decimal
  prices            Price[]
  orders            Order[]
  positions         Position[]
}

model Price {
  id           Int        @id @default(autoincrement())
  instrumentId Int
  instrument   Instrument @relation(fields: [instrumentId], references: [id])
  bid          Decimal
  ask          Decimal
  timestamp    DateTime
}

model Order {
  id           Int       @id @default(autoincrement())
  userId       Int
  user         User      @relation(fields: [userId], references: [id])
  accountId    Int
  account      Account   @relation(fields: [accountId], references: [id])
  instrumentId Int
  instrument   Instrument @relation(fields: [instrumentId], references: [id])
  orderType    OrderType
  side         OrderSide
  quantity     Decimal
  price        Decimal?
  status       OrderStatus
  createdAt    DateTime  @default(now())
  executedAt   DateTime?
  transactions Transaction[]
}

model Position {
  id           Int        @id @default(autoincrement())
  userId       Int
  user         User       @relation(fields: [userId], references: [id])
  accountId    Int
  account      Account    @relation(fields: [accountId], references: [id])
  instrumentId Int
  instrument   Instrument @relation(fields: [instrumentId], references: [id])
  quantity     Decimal
  entryPrice   Decimal
  currentPrice Decimal
  unrealizedPnl Decimal
  openedAt     DateTime
  lastUpdated  DateTime   @updatedAt
  transactions Transaction[]
}

model Transaction {
  id                Int       @id @default(autoincrement())
  accountId         Int
  account           Account   @relation(fields: [accountId], references: [id])
  type              TransactionType
  amount            Decimal
  description       String?
  createdAt         DateTime  @default(now())
  relatedOrderId    Int?
  relatedOrder      Order?    @relation(fields: [relatedOrderId], references: [id])
  relatedPositionId Int?
  relatedPosition   Position? @relation(fields: [relatedPositionId], references: [id])
}

model UserRiskParameter {
  id             Int      @id @default(autoincrement())
  userId         Int      @unique
  user           User     @relation(fields: [userId], references: [id])
  maxLeverage    Decimal
  maxPositionSize Decimal
  maxDailyLoss   Decimal
  lastUpdated    DateTime @updatedAt
}

model MarginUsage {
  id              Int      @id @default(autoincrement())
  accountId       Int
  account         Account  @relation(fields: [accountId], references: [id])
  usedMargin      Decimal
  availableMargin Decimal
  marginLevel     Decimal
  timestamp       DateTime @default(now())
}

model AuditLog {
  id        Int      @id @default(autoincrement())
  userId    Int
  user      User     @relation(fields: [userId], references: [id])
  action    String
  details   String?
  ipAddress String
  timestamp DateTime @default(now())
}

model KYC_AML_Data {
  id                   Int                @id @default(autoincrement())
  userId               Int                @unique
  user                 User               @relation(fields: [userId], references: [id])
  idType               IdType
  idNumber             String
  idExpiryDate         DateTime
  proofOfAddressType   ProofOfAddressType
  proofOfAddressDate   DateTime
  verificationStatus   VerificationStatus @default(PENDING)
  verificationDate     DateTime?
  verifiedBy           Int?
  verifier             User?              @relation("Verifier", fields: [verifiedBy], references: [id])
  additionalNotes      String?
}

enum AccountStatus {
  ACTIVE
  SUSPENDED
  CLOSED
}

enum VerificationStatus {
  UNVERIFIED
  PENDING
  VERIFIED
}

enum AccountType {
  DEMO
  LIVE
}

enum InstrumentType {
  FOREX
  STOCKS
  COMMODITIES
  INDICES
  CRYPTOCURRENCIES
}

enum OrderType {
  MARKET
  LIMIT
  STOP
}

enum OrderSide {
  BUY
  SELL
}

enum OrderStatus {
  PENDING
  EXECUTED
  CANCELLED
  REJECTED
}

enum TransactionType {
  DEPOSIT
  WITHDRAWAL
  TRADE_PROFIT
  TRADE_LOSS
  FEE
  ADJUSTMENT
}

enum IdType {
  PASSPORT
  NATIONAL_ID
  DRIVERS_LICENSE
}

enum ProofOfAddressType {
  UTILITY_BILL
  BANK_STATEMENT
  GOVERNMENT_LETTER
}
