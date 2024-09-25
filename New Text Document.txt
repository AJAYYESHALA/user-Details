package com.example.userdetails.service;

import com.example.userdetails.model.User;
import com.example.userdetails.model.Manager;
import com.example.userdetails.repository.UserJpaRepository;
import com.example.userdetails.repository.ManagerJpaRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import javax.transaction.Transactional;
import java.util.Collections;
import java.util.List;
import java.util.Optional;
import java.util.UUID;

@Service
@Transactional
public class UserService {

    private final UserJpaRepository userRepository;
    private final ManagerJpaRepository managerRepository;

    @Autowired
    public UserService(UserJpaRepository userRepository, ManagerJpaRepository managerRepository) {
        this.userRepository = userRepository;
        this.managerRepository = managerRepository;
    }

    // Create a new user
    public User createUser(User user) {
        user.setUserId(UUID.randomUUID());
        return userRepository.save(user);
    }

    // Get all users or specific user by mobile number or manager
    public List<User> getUsers(Optional<String> mobNum, Optional<UUID> managerId) {
        if (mobNum.isPresent()) {
            return userRepository.findByMobNum(mobNum.get()).map(Collections::singletonList)
                    .orElse(Collections.emptyList());
        } else if (managerId.isPresent()) {
            return userRepository.findByManager_ManagerId(managerId.get());
        }
        return userRepository.findAll();
    }

    // Update a single user
    public User updateUser(UUID userId, User updatedUser) {
        Optional<User> existingUserOpt = userRepository.findById(userId);
        if (existingUserOpt.isPresent()) {
            User existingUser = existingUserOpt.get();
            existingUser.setFullName(updatedUser.getFullName());
            existingUser.setMobNum(updatedUser.getMobNum());
            existingUser.setPanNum(updatedUser.getPanNum());
            existingUser.setManager(updatedUser.getManager());
            return userRepository.save(existingUser);
        } else {
            throw new IllegalArgumentException("User not found with ID: " + userId);
        }
    }

    // Bulk update (Only for manager_id)
    public String bulkUpdateUsers(List<UUID> userIds, UUID newManagerId) {
        Optional<Manager> newManager = managerRepository.findById(newManagerId);
        if (!newManager.isPresent()) {
            throw new IllegalArgumentException("Manager not found with ID: " + newManagerId);
        }

        for (UUID userId : userIds) {
            Optional<User> existingUserOpt = userRepository.findById(userId);
            if (existingUserOpt.isPresent()) {
                User existingUser = existingUserOpt.get();
                existingUser.setManager(newManager.get());
                userRepository.save(existingUser);
            } else {
                throw new IllegalArgumentException("User not found with ID: " + userId);
            }
        }
        return "Bulk update successful";
    }

    // Delete user by ID
    public String deleteUser(UUID userId) {
        if (userRepository.existsById(userId)) {
            userRepository.deleteById(userId);
            return "User deleted successfully.";
        }
        return "User not found.";
    }
}
