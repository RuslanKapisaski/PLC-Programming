# Project 3: Barcode Inventory System

## Description
PLC inventory management system developed in RSLogix 500 and tested with RSLogix Emulate 500.

The program reads barcode data, identifies the product, quantity, and transaction direction, and updates inventory levels.

## Barcode Format
`PPP-QQ:D`

Example: `123-10:1`

- **PPP** – Product ID (123, 456, 789)
- **QQ** – Quantity
- **D** – Direction (1 = Incoming, 2 = Outgoing)



## Example
Input: `123-10:1`

Expected result: `N7:0 = 10`

After processing, the barcode input is cleared to prevent repeated inventory updates.