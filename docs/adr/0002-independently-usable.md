# Keep CloudBarrel independently usable

CloudBarrel is complete on its own and owns backup correctness, History, Integrity, Restore, CLI control, and a stable machine interface. External local applications may consume that interface, but they must not be required to read or restore stored data. This keeps the backup usable without another application or vendor service.
