http://ec2-54-167-103-210.compute-1.amazonaws.com:15672/


http://54.167.103.210:15672/


package com.ust.orderservice.controller;

import com.ust.orderservice.domain.Order;
import com.ust.orderservice.domain.OrderItem;
import com.ust.orderservice.domain.OrderStatus;
import com.ust.orderservice.payload.OrderRequest;
import com.ust.orderservice.service.OrderService;
import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.Collections;
import java.util.List;
import java.util.stream.Collectors;

@RestController
@RequestMapping("/orders")
@RequiredArgsConstructor
public class OrderController {

    private final OrderService orderService;

    // POST /orders create order
    @PostMapping
    public ResponseEntity<Order> createOrder(@RequestBody OrderRequest orderRequest) {
        // Initialize the Order object
        Order order = new Order();
        order.setStatus(OrderStatus.CREATED);

        // Ensure orderItems is not null, initialize it to an empty list if it is
        List<OrderItem> orderItems = orderRequest.orderItems() != null ?
                orderRequest.orderItems().stream().map(orderItem -> {
                    OrderItem item = new OrderItem();
                    item.setSkuCode(orderItem.skuCode());
                    item.setQuantity(orderItem.quantity());
                    item.setOrder(order);  // Reference to the current order
                    return item;
                }).collect(Collectors.toList())
                : Collections.emptyList();

        // Set the order items to the order
        order.setOrderItems(orderItems);

        // Save the order and return the response
        Order savedOrder = orderService.createOrder(order);
        return ResponseEntity.ok(savedOrder);
    }

    // GET /orders/{id} get order by id
    @GetMapping("/{id}")
    public ResponseEntity<Order> getOrderById(@PathVariable Long id) {
        return ResponseEntity.ok(orderService.getOrderById(id));
    }
}
