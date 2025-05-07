# Zephyr: Lack of validation for bluetooth link layer connection



The issue arises in the [ll_tx_mem_enqueue function](https://github.com/zephyrproject-rtos/zephyr/blob/eca57905a3f8c212cfe88f3f2974bcbaa1bf3d20/subsys/bluetooth/controller/ll_sw/ull_conn.c#L229), which only checks if the conn pointer from the connection memory pool is zero before use. However, the conn structure members may not be fully initialized, risking invalid pointer dereference.



To be specific,  [`hci_acl_handle` function](https://github.com/zephyrproject-rtos/zephyr/blob/02f85b253a16f71176d8e4995d801ac464a14633/subsys/bluetooth/controller/hci/hci.c#L5856-L5858), which is called by bluetooth send function `bt_send`, extracts the acl->handle from an untrusted buffer. The acl->handler member is used to locate a connection pointer conn from the conn_pool memory pool in [`ll_tx_mem_enqueue` function](https://github.com/zephyrproject-rtos/zephyr/blob/eca57905a3f8c212cfe88f3f2974bcbaa1bf3d20/subsys/bluetooth/controller/ll_sw/ull_conn.c#L228). However, pointers in conn_pool is not be fully initialized in [`mem_init` function](https://github.com/zephyrproject-rtos/zephyr/blob/eca57905a3f8c212cfe88f3f2974bcbaa1bf3d20/subsys/bluetooth/controller/ll_sw/ull_conn.c#L1664-L1668), allowing an attacker to manipulate the indicator, leading to an uninitialized pointer conn member, as seen in the `ull_conn_tx_lll_enqueue` function.




We validated this problem using the Zephyr HCI-UART example on the nrf52840dk board and has reported to zephyr community and opened a patch to fix it https://github.com/zephyrproject-rtos/zephyr/pull/89481.

