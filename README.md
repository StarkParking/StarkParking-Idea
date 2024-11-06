# Project Idea: StarkParking

## Introduction

StarkParking is a parking application built on the Starknet platform, allowing users to reserve spots and make payments using the `STRK` token. The application will store all related data, including parking lots, bookings, and parking times, on the blockchain to ensure transparency and security.

## Key Features

### 1. Parking Lot Registration

- Users can register parking lots by paying a fee, which will be refunded after a specified period.
- The creator must provide a wallet address for receiving `STRK` tokens when drivers make payments.

### 2. Parking Spot Reservation

- Users can select a parking lot and slot, then record their entry time.
- The system will check the slot status before allowing reservations.

### 3. Service Fee Calculation

- Prices will be set in `cents (USD)` and pegged to the value of `USDT`.
- An oracle will be used to convert the price from USD to the equivalent amount in `STRK`.

### 4. Telegram Mini App Integration

- Users can make reservations and payments via a Telegram Mini App.
- The app sends notifications via Telegram Bot when the booking time is about to expire, allowing users to extend their booking time.
- Parking lot managers can verify the license plate of parked vehicles against the reservation.
- Correctly parked vehicles can leave, while mismatched vehicles face penalties.

### 5. License Plate Validity Check
- Parking lot managers can check if a vehicle’s license plate is valid for a given parking lot.
- If the license plate is found to be invalid, the manager can impose penalties, ensuring compliance with parking regulations.

### 6. Data Storage

- All information about parking lots and bookings will be stored on the blockchain to ensure transparency and security.

## Data Structures

### 1. Parking Lot Data Structure

```rust
struct ParkingLot {
    lot_id: u256, // Associated parking lot
    name: felt252,
    location: felt252,
    coordinates: felt252,
    slot_count: u32,
    hourly_rate_usd_cents: u32,
    creator: ContractAddress,
    wallet_address: ContractAddress,
    is_active: bool,
    registration_time: u64
}
```

### 2. Booking Data Structure

```rust
struct Booking {
    license_plate: felt252, // Vehicle license plate number
    booking_id: felt252, // Unique identifier for the booking
    lot_id: u256, // Associated parking lot
    entry_time: u64, // Timestamp of entry
    exit_time: u64, // Timestamp of exit
    expiration_time: u64, // Timestamp indicating when the booking expires
    total_payment: u64, // Total payment amount in cents
    payer: ContractAddress // Wallet address of the user
}
```

### Define the Storage

```rust
#[storage]
struct Storage {
    parking_lots: Map::<u256, ParkingLot>, // Mapping from lot_id to ParkingLot
    bookings: Map::<felt252, Booking>, // Mapping from booking_id to Booking
    available_slots: Map::<u256, u32>, // Mapping from lot_id to available slots
    payment_token: ContractAddress, // TODO: remove it
    license_plate_to_booking: Map::<felt252, felt252> // License_plate to booking_id
    ...
    ...
}
```

## Interface

```rust
#[starknet::interface]
pub trait IParking<TContractState> {
    // Register a new parking lot
    fn register_parking_lot(
        ref self: TContractState,
        lot_id: u256,
        name: felt252,
        location: felt252,
        coordinates: felt252,
        slot_count: u32,
        hourly_rate_usd_cents: u32, // 100 = $1
        wallet_address: ContractAddress
    );

    // Create a booking for a parking spot
    fn book_parking(
        ref self: TContractState,
        booking_id: felt252,
        lot_id: u256,
        payment_token: ContractAddress,
        license_plate: felt252,
        duration: u32, // Duration in hours
    );

    // End a parking session
    fn end_parking(ref self: TContractState, booking_id: felt252);

    // Extend a parking session
    fn extend_parking(
        ref self: TContractState,
        booking_id: felt252,
        additional_hours: u32,
        payment_token: ContractAddress
    );

    // Get available slots in a parking lot
    fn get_available_slots(self: @TContractState, lot_id: u256) -> u32;

    // Validate if the vehicle license plate is valid for the given lot
    fn validate_license_plate(self: @TContractState, lot_id: u256, license_plate: felt252) -> bool;
    ...
    ...
}
```
